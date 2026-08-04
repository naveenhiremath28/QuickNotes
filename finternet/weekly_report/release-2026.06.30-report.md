# Release 2026.06.30 — "Federation Convergence"

> Report generated 2026-07-07. Covers the coordinated release train merged to `main` on July 6, 2026 (release PRs authored by @yravinderkumar33 as release manager).

**Release PRs:**

| Repo | PR | Files | Diff |
|---|---|---|---|
| finternet-app | [#368](https://github.com/finternet-io/finternet-app/pull/368) | 239 | +11,545 / −4,718 |
| units-api | [#304](https://github.com/finternet-io/units-api/pull/304) | 285 | +40,846 / −18,011 |
| units-services | [#166](https://github.com/finternet-io/units-services/pull/166) | 146 | +19,725 / −257 |
| units-token-runtime | [#139](https://github.com/finternet-io/units-token-runtime/pull/139) | 165 | +14,407 / −3,634 |
| units-workflows | [#113](https://github.com/finternet-io/units-workflows/pull/113) | 62 | +9,563 / −6,338 |
| wpbe-silencelabs | [#23](https://github.com/finternet-io/wpbe-silencelabs/pull/23) | 6 | +194 / −124 |

This was one coordinated release across 6 repos, re-architecting the platform around Finternet federation: registry-anchored identity, OTP-first signup (ADR-007), the primitive-ops pipeline as the sole backend path, and UUID-based instance identity.

---

## Release notes by repo

### 1. finternet-app #368

- **Added:** Federation primitive-ops consolidation (#321); OTP-first federated signup per ADR-007 with registry name reservation and MPC/passkey removed from signup (#324); JWT-driven instance routing via a new `@finternet/instance-routing` package resolving the `instance_url` claim (#349); scroll-gated Terms & Privacy consent UI with a re-consent `AuthGuard` modal (#355); registry-anchored Google SSO (#365); async transfer with an inline OTP "Verify to sign" fallback for users without an MPC key (#328); redesigned Transfer page and 7-column sortable transaction list with counterparty names (#364).
- **Changed:** Super-admin UI keyed off `is_super_admin` (#348); holder-balance display everywhere instead of total supply, `amount` → `value` on the wire (#328); Docker build moved to `next start` (#344).
- **Fixed:** 10+ token-program-refactor review findings (#344); `terms/get` 401-on-every-call routing bug (#366); signup consent double-prompt (#367).
- **Removed:** MPC/passkey keygen and client-side reservation proof from signup; static units-api URL config.

### 2. units-api #304 (largest of the release)

- **Added:** Server-side name reservation + OTP-first synchronous signup with Vault Transit Ed25519 keys and ULIP Go SDK (#219); capability publisher + REST registry client with pinned-key verification and JCS via `json-canon` (#104/#186); Terms & Privacy versioning/consent ledger with `RequireConsent()` middleware shipped dark (#296); super-admin cross-owner scope approvals (#286); plan-declared audit pruning + `PruneTokenTransactions` peer endpoint (#299); registry-authoritative dev credentials + Vault-transit owner-signing and cross-instance delegation (#232); FT freeze/unfreeze/update over federation (#283).
- **Changed:** Federated account name as first-class identity + transaction counterparty enrichment (#301/#295); synchronous signup registration + UUID/`issuer_url` identity migration (#285); transaction identity model alignment and `amount` → `value` payload standardization with the `{ key_id, jws }` envelope (#234); ULIP primitive `amount` → `value` canonicalization (#229/#222); DB schema cleanup with `plan_data JSONB` consolidation (#236); OTP step-up removed from owner-signed mutations (#284); synchronous cross-instance scope approve (#303).
- **Fixed:** RS256 + EdDSA SSO JWT verification (#302); 28 token-program-refactor review findings (#278); the `primitive_plan_intent_hash_mismatch` transfer bug (#301/#295).
- **Removed:** `enable_federation` flag — federation is now the only path (#283); scope-approval Restate workflow; `token_pools`, `primitive_templates`, finalize callback.

### 3. units-services #166

- **Added:** New central name & capability **registry sub-service** in Go (#69); registry-authoritative developer-credential store (#125); registry-anchored Google SSO in otp-service (#162); EdDSA support + `/.well-known/ulip.json` (#116); cross-owner pending-scope listing (#154); scope-expiry sweeper (#165); registry CI/build pipelines (#109); registry OpenTelemetry instrumentation (#153).
- **Changed:** UUID instance identity (#155); `registered_names` merged into a unified `accounts` table with signed names (#161); authoritative signing-key storage (#126); dual-alg OTP-JWT verification (#164); mandatory OTP challenge verification (#109).
- **Fixed:** 17 review findings (#150); account-create NULL-UUID 500 (#163); proofService prune race via row-locking (#159); capability-document signature mismatch (#109).
- **Removed:** Plaintext email/mobile from registry accounts — hashed contacts only (#161).

### 4. units-token-runtime #139

- **Added:** PLAN-5 federation token primitives (lock/commit/compensate) on a single `primitive_ops` table (#94); `OperationPlan` + `plan()` on the `TokenProgram` trait with authoritative `state.balance` (#108); Ed25519 ULIP envelope verification (#108/#110); federation-only `domain_lifecycle` incl. NFT `local_transfer` (#129/#137/#138).
- **Changed:** `amount` → `value` everywhere (#109); issuer/holder state split fixing the balance math (#109); resolved-verb `token_transactions` audit model (#137); signer instance from `LOCAL_INSTANCE_ID` (#130); reference programs renamed to canonical names (`fungible`, `non-fungible`, …).
- **Fixed:** FT burn rewired onto `state.balance` (#128); spendable-balance read fix so federation-received funds are spendable (#111); Active-status gating on value-moving primitives (#128); proxy import shadow-token corruption (#129); federated recipient transaction row (#138); idempotency/authorization hardening (#128).
- **Removed:** `token_pools`, the audit-gate env, and the legacy single-message transfer path.

### 5. units-workflows #113

- **Added:** Generic primitive-operation Restate workflow with saga compensation, replacing the hardcoded transfer workflow (#80); `RegistryClient` with Ed25519 JWS verification (#35); ULIP TypeScript SDK (#88); best-effort post-commit prune of superseded prepare rows (#110).
- **Changed:** Registry-authoritative delegation/scope-approval (#92); JCS library swap + name folded into the signed record (#35/#111); **BREAKING** — `LOCAL_INSTANCE_NAMESPACE` removed, `LOCAL_INSTANCE_ID` (UUID) now required (#111/#108); `value` forwarded in step payloads (#90).
- **Fixed:** 11 review findings incl. a fail-closed reconcile fan-out that retried forever (#106).
- **Removed:** The Restate **signup** workflow (#108/#111) and the **scope-approval** workflow (#112).

### 6. wpbe-silencelabs #23

- Added `ChainType::Finternet` — raw 64-byte Ed25519 signing over JCS canonical bytes for the federation identity flow; migrated Silence Labs crate sourcing from SSH/git to the Kellnr sparse-HTTPS registry with token-only auth; restored `git` in the Docker builder for `build.rs`.

---

## My contribution (@naveenhiremath28)

### Features authored that shipped in this release

**finternet-app** — biggest surface, 3 of the release's 7 headline features:

- **#349 — JWT-driven instance routing**: the new `@finternet/instance-routing` package and the anti-SSRF, fail-closed proxy routing off the `instance_url` claim.
- **#328 — Async transfer with OTP-sign fallback** + holder-balance display across dashboard/holdings/details.
- **#364 — Transfer page & transaction list redesign** (receipt-style summary, counterparty names, sortable table, Recent Activity).

**units-api** — 4 cited PRs:

- **#301 / #295 — Federated account name + transaction counterparty** (incl. the `primitive_plan_intent_hash_mismatch` fix).
- **#285 — Synchronous signup name registration** (replacing the async Restate finalize callback).
- **#234 — Transaction identity model alignment + `amount` → `value` / `{ key_id, jws }` envelope standardization**.

**units-services** — 5 cited PRs:

- **#161 — Merge `registered_names` into unified `accounts`** with signed names and hashed-contacts-only privacy.
- **#155 — UUID instance identity migration** (drop `issuer_url`).
- **#153 — Registry OpenTelemetry instrumentation**.
- **#109 — Registry CI/build pipelines + capability-signature fix + OTP challenge hardening**, plus **#127** (.env.example refresh).

**units-token-runtime** — 3 cited PRs: **#138** (record all token operations, federated recipient row fix), **#130** (signer from `LOCAL_INSTANCE_ID`), **#110** (ULIP envelope verification aligned to value-keyed payload).

**units-workflows** — 2 cited PRs: **#111** (JCS signed-name + UUID identity, the release's one BREAKING change) and **#108** (decommission the signup workflow).

**wpbe-silencelabs** — no contribution.

The theme: I owned the **identity-model thread** of the federation release — UUID instance identity, signed account names, counterparty enrichment, and the `amount` → `value` standardization — end-to-end across five repos, plus the transfer UX in the app.

### Contribution in percentage (by commits in the release PRs)

| Repo | My commits | Total commits | My share |
|---|---|---|---|
| finternet-app #368 | 46 | 100 | **46.0%** (top contributor) |
| units-services #166 | 36 | 100 | **36.0%** (top contributor) |
| units-workflows #113 | 10 | 67 | 14.9% |
| units-token-runtime #139 | 9 | 73 | 12.3% |
| units-api #304 | 5 | 100 | 5.0% |
| wpbe-silencelabs #23 | 0 | 8 | 0% |
| **Overall release** | **106** | **448** | **≈ 23.7%** |

By cited feature PRs the share is similar: 17 of the ~70 PRs called out in the release notes (~24%).

---

## Overall team contribution

Across all 6 release PRs (448 total commits):

| Contributor | Commits | Share |
|---|---|---|
| @ravismula | 182 | **40.6%** |
| @yravinderkumar33 | 149 | **33.3%** |
| @naveenhiremath28 (me) | 106 | **23.7%** |
| @LikhitaVadapally | 10 | **2.2%** |
| @szymek156 | 1 | **0.2%** |

Per-repo breakdown:

| Repo (commits) | ravismula | yravinderkumar33 | naveenhiremath28 | LikhitaVadapally | szymek156 |
|---|---|---|---|---|---|
| finternet-app (100) | 21% | 27% | **46%** | 6% | — |
| units-api (100) | **72%** | 23% | 5% | — | — |
| units-services (100) | 25% | 35% | **36%** | 4% | — |
| units-token-runtime (73) | **49%** | 38% | 12% | — | — |
| units-workflows (67) | 37% | **48%** | 15% | — | — |
| wpbe-silencelabs (8) | 38% | **50%** | — | — | 12% |

The pattern: @ravismula dominated the backend core (units-api, token runtime), @yravinderkumar33 led the workflows and WPBE plus release management across all repos, and @naveenhiremath28 led the frontend/BFF (finternet-app) and units-services (registry sub-service work). Each of the top three contributors was #1 in two of the six repos.

**Caveat:** shares are by commit count, which doesn't weight commit size — by lines changed, units-api's +40k/−18k diff would tilt the totals further toward @ravismula.
