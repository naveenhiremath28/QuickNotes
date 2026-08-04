# units-workflows · wpbe-silencelabs · units-mcp · Local Dev Stack

## units-workflows — Restate Durable Workflows

**Path:** `units-workflows/` · **Stack:** TypeScript, npm workspaces + Turborepo, `@restatedev/restate-sdk`, Zod, Prisma, Node ≥ 24

> ⚠️ The sub-project CLAUDE.md is stale: reality is **4 live apps + 1 dead** and **9 packages**, not 2 apps / 5 packages.

### Workspace

| App | Restate service(s) | Port | Role |
|---|---|---|---|
| `apps/gateway` | binds all workflows | 9080 | **The deployable** — imports every workflow, runs all registrations, serves one Restate HTTP/2 (h2c) endpoint |
| `apps/profile-update` | `profileUpdate` | 9080 | Profile update with OTP verification |
| `apps/delegation` | `delegation`, `delegation-revoke` | 9081 | RBAC delegation grant/approval + revoke |
| `apps/primitive-operation` | `primitive-operation` | 9084 | Generic plan-driven federated saga executor |
| `apps/scope-approval` | — | — | **Vestigial** — only a `.env`, no source; still referenced by `register-restate.sh` |

Packages: `@units/restate-utils` (graceful h2c endpoint + `awaitPromiseWithTimeout`), `@units/workflow-db` (Prisma for `workflows`/`workflow_ops`), `@units/db` (a **second** Prisma client mirroring units-api's schema — label resolver, `reconcileIdentities`), `@units/workflow-utils` (signed ULIP client, retry, JCS), `@units/http-client` (`{context, payload}` envelope), `@units/otel`, `@units/logger`, `@units/validators`, `@units/typescript-config`.

### The four workflows

**`profileUpdate`** — handlers `run`, `verify`, `cancel`, `getStatus`. Validates → creates workflow record → loops over units-api-precomputed `verificationSteps` (each: call notificationService `POST /api/v1/otp/generate`, then await durable promise `verify-current-ownership` / `verify-new-email` / `verify-new-mobile` with OTP timeout 10 min) → executes the update via `POST /v1/workflows/execute` (`action: execute_update`, user JWT forwarded) → completed. `cancel` rejects all three promises.

**`delegation`** — handlers `run`, `approve`, `reject`, `cancel`, `getStatus`. **Registry-authoritative: there is no local `delegations` table** — the central registry row is the truth, flipped via signed ULIP calls (`DelegationApprove`/`DelegationReject`). Flow: validate → validate label against the Prisma-DMMF schema mirror (failure → signed `DelegationReject`, early return) → owner path activates immediately; non-owner path awaits `owner-approval` durable promise (default timeout **7 days**). Activation = signed ULIP `DelegationApprove` to the registry, then **fail-closed `fanOutReconcile`**: `POST {peer}/v1/internal/delegation/reconcile` to *every* peer in `RECONCILE_PEER_URLS` (including the local instance); any peer failure throws non-terminal so Restate retries until every host restamps `identities[]` (a half-applied grant would make OPA Rule-3 access inconsistent). Terminal branches match **exact** error messages (cancelled/rejected/timeout-expired) to avoid misclassifying a Prisma failure as a timeout.

**`delegation-revoke`** — separate workflow (no approval wait): validate → label check → signed ULIP `DelegationRevoke` → same fail-closed reconcile fan-out. The reconcile is a full formula re-run, so revoking a *deny* rule can restore a previously-shadowed allow.

**`primitive-operation`** — handlers `run`, `primitiveCompleted`, `getStatus`. A **template-agnostic saga engine** executing the versioned `PrimitivePlan` snapshot produced by units-api's operation planner (`routeSnapshot`, `participants`, `assetSelector`, `steps[]` with `opSeq`/`method`/`dependsOn`/`proofRefs`/`prunesOpSeq`/`retryPolicy`, `compensationPolicy`, `statusPolicy`). Per step: build payload (flatten assetSelector role-aware, resolve `$.`-path + proof refs fail-loud, strip reserved namespaces) → send signed ULIP envelope `POST {endpoint}/v1/ulip/{method}` (with `commitWithRetry` when declared) → record audit op (`/v1/internal/workflow-ops/record`, envelope bytes SHA-256 hashed) → await durable promise `primitive:{txnId}:{opSeq}` (default 30s) → verify completion method matches → prune superseded prepare rows (best-effort, tight 15s retry budget, never throws) → persist status (+`/v1/internal/transactions/lifecycle-update`).

Failure handling around `pointOfNoReturnOpSeq`: before PONR → run `prepareFailure` compensations → `ABORTED`; pre-dispatch 4xx after PONR → compensate directly → `REVERSED` (destination definitively didn't commit); genuinely uncertain after PONR → query `/v1/internal/transactions/destination-status`: `COMMITTED` → committed, `NOT_COMMITTED` → compensate → `REVERSED`, `UNKNOWN` → **`STUCK`**. `primitiveCompleted` is idempotent (peeks the promise, absorbs duplicate signals from the outbox race). `planHash` = SHA-256 over a Go-`omitempty`-byte-compatible projection — a hard cross-language contract (mismatch = `primitive_plan_intent_hash_mismatch`).

### Invocation & callbacks

units-api → Restate ingress (`restate_client.go`): `StartWorkflow` (`/run/send`), `RunWorkflowSync` (`/run`, blocking), `SignalWorkflow` (`/{handler}/send`), `QueryWorkflow`, `CancelWorkflow`. Generic dispatch via `POST /v1/workflows/execute` driven by the `workflow_registry` table (first-step name → start, else signal; registered domain handlers take full control). `primitive-operation`'s workflow ID **is the `txnId`** (others use `{name}-{uuid}` — status lookup depends on that convention).

Workflows call back with `{context, payload}` envelopes: self-registration at startup (`/v1/internal/workflows/register`, retried 5×; total failure = `process.exit(1)`), delegation reconcile fan-out, primitive audit/lifecycle/destination-status, and signed ULIP `POST {endpoint}/v1/ulip/{method}` calls. The gateway also self-registers with the Restate admin (`POST /deployments`, idempotent).

**ULIP signing contract:** signing input = `JCS(canonical_context) ‖ JCS(payload)` (snake_case, signature fields excluded), Ed25519, byte-compatible with units-api's Go implementation, pinned by a shared golden test corpus. Error mapping: 4xx → `TerminalError` (stop), 5xx/network/non-JSON 2xx → retryable Error. Callee idempotency key: `(callerInstance, txnId, opSeq)`.

### Config highlights
Per-app Zod-validated env (`DATABASE_URL`, `WORKFLOW_API_URL`, `DEVELOPER_TOKEN`, `OTP_SERVICE_URL`, `REGISTRY_BASE_URL`, `REGISTRAR_NAMESPACE`, `LOCAL_INSTANCE_ID`, `RECONCILE_PEER_URLS`, `ULIP_SIGNING_PRIVATE_KEY/KEY_ID`). Guard: non-empty `RECONCILE_PEER_URLS` **must** include `LOCAL_INSTANCE_ID` or grants silently half-apply.

### Known drift
`specs/db/workflows.sql` still has the pre-federation shape (Prisma migrations are the real truth). Three conflicting port maps (Zod defaults vs `register-restate.sh` vs docker-compose). Local dev: `postgres:18-alpine` on 5433, `restatedev/restate:1.3` (ingress 8082, admin 9070).

---

## wpbe-silencelabs — MPC Wallet Provider Backend

**Path:** `wpbe-silencelabs/` · **Stack:** Rust, Axum 0.8 + Tower, Silence Labs private-registry SDK (`el-aggregator-client`, `wallet-provider-service-sdk`, `remote-attestation`)
**Workspace:** single crate `crates/wallet-provider-service` · **Port:** `PORT`/`LISTEN` env (default 8090)

### Purpose
The **"Wallet Provider" party** in Silence Labs' threshold-MPC custody protocol — a **stateless relay/orchestrator** that lets Finternet clients generate and use MPC wallet keys over REST/WebSocket without any single party ever holding a full private key. All threshold cryptography and keyshare custody live inside the closed-source SDK and the remote **Silent Network operator nodes** (3 in sandbox, WSS + GCP Confidential Space attestation). WPBE's only private material is its own identity key (`WPBE_PRIVATE_KEY`); **keyshares never touch this process, and there is no database** (the Helm `DB_*` config and pg scripts are unused scaffolding — zero DB deps in Cargo.lock).

### API surface (7 routes)

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | Probe target, returns `"ok"` |
| GET | `/v2/version` | Build/version/features/public key |
| POST | `/v2/rest/keygen` | Threshold DKG against SN nodes → per-key `{key_id, address, sign_alg, public_key, threshold, total_parties}`. Address derivation: SECP256k1 → Ethereum address; ED25519 → base58 (Solana) |
| POST | `/v2/rest/signgen` | Threshold signing. Body enforces chain/alg consistency at parse time (ethereum⇒SECP256k1, solana⇒ED25519). Ethereum output `0x r‖s‖(recid+27)`; Solana pass-through |
| POST | `/v2/rest/linkCredentialToKeyshare` | Bind a WebAuthn passkey / JWT subject to an MPC key (authorization policy for signgen) |
| POST | `/v2/rest/destroyLinkCredentialToKeyshare` | Unbind (device loss / rotation) |
| WS | `/v1/registerPasskey` | Full WebAuthn registration ceremony over the socket (SDK-driven) — this is what the finternet-app BFF proxies |

Requests/responses use the `{context, payload}` / `{context, response}` envelope; `context` is opaque and echoed back.

### Cross-cutting notes
- **No visible inbound auth** — auth features (`passkey-auth`, `jwt-auth`) gate route compilation and SDK flags; enforcement, if any, is inside the opaque SDK. Trust boundary as-written: network reachability.
- Tower stack on every route (incl. probes): concurrency limit 100, 50s timeout → 408, load-shed → 503.
- Inbound TLS not terminated here (rustls is for outbound WSS); errors collapse to generic 500 strings.
- ⚠️ Helm models `API_KEY`/`WPBE_PRIVATE_KEY`/`DB_PASS` as **ConfigMap** values, not Secrets. Dockerfile HEALTHCHECK hits `/version` (route is `/v2/version` — 404s). OTel deps present but no exporter wired.

---

## units-mcp — MCP Server for AI Agents

**Path:** `units-mcp/` · **Stack:** TypeScript ESM, `@modelcontextprotocol/sdk`, Express 5, Zod · **Port:** `PORT` (default 3000)

**Streamable HTTP transport with stateful sessions:** `POST /mcp` (initialize creates a per-session `McpServer`; `mcp-session-id` header routes), `GET /mcp` (SSE), `DELETE /mcp` (terminate), `GET /health`. Sessions in an in-process Map. **No server-side auth on `/mcp` itself** — the per-session developer token (set via `units_login`, held in memory only) is what gates the downstream UNITS API. All calls wrap the standard `{context, payload}` envelope to `{UNITS_API_BASE}/v1/...` (30s timeout).

### Tools (10 — all UNITS-facing tools are read-only)

| Tool | UNITS endpoint |
|---|---|
| `units_login` | local — stores the `fnt_…` developer token in session |
| `units_logout` | local — clears token and closes the session |
| `units_get_account` | `POST /v1/account/get` |
| `units_search_tokens` | `POST /v1/token/search` (limit/offset pagination) |
| `units_get_token` | `POST /v1/token/get` |
| `units_search_token_classes` | `POST /v1/tokenclass/search` |
| `units_get_token_class` | `POST /v1/tokenclass/get` |
| `units_get_transaction` | `POST /v1/transaction/get` |
| `units_get_transaction_status` | `POST /v1/transaction/status` |
| `units_search_transactions` | `POST /v1/transaction/search` |

One MCP resource: `units://api/overview` (markdown overview of the wrapped endpoints).

> ⚠️ `units-mcp/readme.md` is badly stale (documents 13 tools incl. mint/transact, unused env vars, "stateless" transport). Trust the code: 10 read-only tools, stateful sessions. Config: `PORT`, `CORS_ORIGIN`, `UNITS_API_BASE` (bare host — `/v1` appended in code).

---

## finternet-services — Local Dev Stack (Docker Compose)

Two compose files: `manifests/units.yaml` (backend, profile-gated) + `manifests/app.yaml` (frontend). `make start` brings up profiles `api-infra`, `otp`, `app`, `kafka`, `engine`.

| Service | Image | Host ports | Profile |
|---|---|---|---|
| `db` | postgres:18-alpine | 5432 | always |
| `hyperdx` | clickhouse/clickstack-local | 8081 (UI), 4317/4318 (OTLP) | always |
| `minio` | minio/minio | 9005, 9001 | api-infra |
| `keycloak` | keycloak:26.4.1 (`--import-realm`) | 127.0.0.1:8080, :9000 | api-infra |
| `vault` | hashicorp/vault | 8200 | api-infra |
| `otp-service` | ghcr.io …/otp-service | 4000 | otp |
| `app` (units-api) | ghcr.io …/api | 3000 | app |
| `kafka` | apache/kafka:4.1.1 (**KRaft**) | 9092, 29092 | kafka |
| `kafka-init` | apache/kafka | — | kafka — creates `units.token.operations` (4 partitions), `…dlq` (1), `units.token.events` (4) |
| `token-engine` | ghcr.io …/token-engine | (internal :8080 health) | engine |
| `finternet-app` | ghcr.io …/finternet-app/app | 3002 | app.yaml |

Restate + workflow services are **not** in this stack — they come from `units-workflows/docker-compose.yml` (ingress 8082, admin 9070) and reach units-api/OTP via `host.docker.internal`.
