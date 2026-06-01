# Spec 01 — ULIP Protocol — Plain English Walkthrough

> Source: `units-federation/01-ulip-protocol.md`
> Owner: Protocols team. Foundational — depends on nothing, consumed by Specs 02, 03, 04, 05, 06.

---

## 1. The one-line summary

**ULIP** = the **shared language** every UNITS instance must speak to talk to any other UNITS instance. Without it, instances can't say anything to each other.

Full name: **Unified Interledger Protocol** (think "SWIFT for tokenized value, but cryptographically verifiable").

---

## 2. The mental model — a tamper-evident FedEx package

Picture sending a **registered courier package**:

| Real-world                                   | ULIP                                                                        |
| -------------------------------------------- | --------------------------------------------------------------------------- |
| Outer label (FROM, TO, tracking #, postmark) | **Context** — who's calling, who's being called, when, what method          |
| The actual stuff inside the box              | **Payload** — the request/response body                                     |
| Tamper-evident wax seal                      | **Signature** — proves the sender wrote it + nothing was changed in transit |
| FedEx delivery network                       | **gRPC or REST transport**                                                  |
| The signed receipt you keep                  | **Cached response** (for replay protection)                                 |

The whole package is called the **envelope**. Every conversation between instances is a stack of these envelopes.

---

## 3. Why it's its own repo (`units-ulip/`)

The other 7 services live in one monorepo. **ULIP gets its own repo**, sibling to the rest. Three reasons:

1. **External operators** (other companies running their own UNITS instance) need the protocol but shouldn't have to pull in our entire monorepo.
2. The protocol gets its **own release cadence** — independent from internal service changes.
3. **Everything depends on the published SDKs** (Go, Rust, TypeScript), not on local protocol source. This forces a clean contract boundary.

The repo ships:
- `.proto` files (the wire format)
- 3 SDKs: Go, Rust, TypeScript
- Canonical serialisation rules (more on this below)
- Conformance test harness (anyone implementing ULIP can verify their impl)
- OpenAPI auto-generated from `.proto` (for REST clients)

---

## 4. The envelope — the heart of ULIP

Every message has **three sections**:

```
┌──────────────────────────────────────────────┐
│  CONTEXT (the outer label)                    │
│  - ulip_version: "v1"                         │
│  - method: "Lock" / "CreateIncoming" / ...    │
│  - txn_id: workflow transaction ID            │
│  - op_seq: 0, 1, 2, ... (step number)         │
│  - caller_instance: "a.finternet.global"      │
│  - callee_instance: "b.finternet.global"      │
│  - sent_at: timestamp                         │
│  - trace_id: for distributed tracing          │
├──────────────────────────────────────────────┤
│  PAYLOAD (the contents)                       │
│  - oneof body { LockRequest, CreditRequest,   │
│                 RegisterNameRequest, ... }     │
├──────────────────────────────────────────────┤
│  SIGNATURE (the wax seal)                     │
│  - signer_instance: who signed                │
│  - key_id: which key                          │
│  - algorithm: "ed25519" or "ecdsa-p256"       │
│  - signature: bytes signed over context+payload │
└──────────────────────────────────────────────┘
```

### Why each piece matters

- **`txn_id` + `op_seq`** — together they are the **idempotency key**. Same `(txn_id, op_seq)` → same response, no re-execution. This is what makes retries safe.
- **`caller_instance` / `callee_instance`** — explicit "I am A talking to B"; receiver can reject if the caller pretends to be someone else.
- **`sent_at`** — replay protection lives in the (caller, txn_id, op_seq) cache for ≥ 7 days, but timestamp helps debug clock skew.
- **`extensions` (map)** — escape hatch for forward-compat. Reserved field `delegated_capability` is here.
- **Signature** — proves the sender wrote it AND nothing was edited in transit.

---

## 5. The trickiest concept: **canonical serialisation**

This is the part that's easy to misunderstand.
 
**Problem**: I sign a JSON object. You receive it. You parse it. You re-serialise it. The bytes you re-serialised **don't match** the bytes I signed (different key order, different whitespace, different number formatting). Verification fails.

**Solution**: both sides agree on **EXACT byte-for-byte rules** for how to lay out the message. This is "canonical serialisation."

Two transports → two canonical formats, **same signature**:
- **gRPC / Protobuf** — deterministic encoding: sort map keys, no unknown fields, fixed field order. (Rust: `prost` canonical flag; Go: `proto.MarshalOptions{Deterministic: true}`.)
- **REST / JSON** — **RFC 8785** (JSON Canonicalization Scheme).

> Whether you send the message as Protobuf bytes or as JSON, the **signed bytes are identical**. That's why "Go signer → Rust verifier" works (test T-01-8 specifically asserts this).

---

## 6. The method catalog — the "menu" instances can call

ULIP defines a **fixed set of methods** instances can call on each other. Grouped:

| Group                   | Methods                                                                                                | What they do                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| **Discovery**           | `Info`, `Capabilities`, `GetAccount`                                                                   | "Who are you, what do you support, do you host this account?"               |
| **Registry**            | `RegisterName`, `ResolveName`                                                                          | Claim a name; look one up                                                   |
| **Transfer primitives** | `Lock`, `CommitDebit`, `Unlock`, `CreateIncoming`, `CommitCredit`, `RejectIncoming`, `Credit`, `Debit` | The atomic moves a transfer is built from (Spec 02 defines these in detail) |
| **Supply**              | `Mint`, `Burn`                                                                                         | Issuer-only                                                                 |
| **Token class**         | `GetTokenClass`, `SubscribeTokenClass`                                                                 | Look up / subscribe to token definitions                                    |
| **Migration**           | `InitiateMigration`, `AcceptMigration`, `FinalizeMigration`                                            | Move an account between instances (Spec 05)                                 |
| **Proxy view**          | `RecordProxyEntry`                                                                                     | Mirror an off-chain swap on peer instances                                  |


For each method the spec records: **idempotent? state-changing? does it need `op_seq`?** Read-only methods (`Info`, `ResolveName`) don't need `op_seq` because they don't change state.

---

## 7. Security — three layers stacked on top of each other

### Layer 1 — Transport (mTLS)
- Both sides present TLS certificates. **The certificate IS the instance identity.**
- gRPC preferred; REST gateway available with the same envelope.
- mTLS prevents man-in-the-middle attacks.

### Layer 2 — Envelope signature
- Signed over `canonical(context) || canonical(payload)`.
- Algorithms: `ed25519` or `ecdsa-p256`.
- Even if transport is compromised, the signature proves authenticity.
- The signed envelope is the **durable, replayable evidence** — it can be stored, audited, replayed later.

### Layer 3 — Replay protection (the `(caller, txn_id, op_seq)` cache)
- Server **must cache the response** for ≥ 7 days (default).
- **Same `(caller, txn_id, op_seq)` → return the cached response,** **do NOT re-execute**.
- This is what makes retries safe AND prevents attackers replaying old messages.

### The verifier algorithm (5 steps)
1. Check `ulip_version == "v1"`.
2. Look up the signer's public key (cached, or fetch via `Info`).
3. Compute `digest = SHA-256(canonical(context) || canonical(payload))`.
4. Verify the signature.
5. Reject if `caller_instance != signer_instance` (unless using future delegated capability).
6. Check the idempotency cache — if seen, return the cached response.

---

## 8. Error codes (the canonical list)

| Code | Meaning |
|---|---|
| `UNAUTHENTICATED` | Bad signature / unknown signer |
| `PEER_DENIED` | Caller not in callee's allowlist |
| `VERSION_UNSUPPORTED` | Wrong `ulip_version` |
| `OP_SEQ_OUT_OF_ORDER` | Referenced an `op_seq` we haven't seen |
| `IDEMPOTENT_REPLAY` | Already seen — returning cached response |
| `INSUFFICIENT_BALANCE` | Reversal/debit fails balance check |
| `LOCK_TIMEOUT_EXPIRED` | The `Lock` referenced has expired |
| `ACCOUNT_FROZEN` | Account is frozen — refused |
| `TOKEN_CLASS_REJECTED` | This instance doesn't host this class (jurisdictional) |
| `FORWARD` | Account migrated; `details.new_home` has the new namespace |
| `INTERNAL` | Unexpected error |

These error codes are **stable wire contract** — clients in any language can pattern-match on them.

---

## 9. Versioning — why "additive only" within v1

The rules:
- Major version in `context.ulip_version` AND in the URL/service name.
- **Within v1**: additive only. New fields get new tag numbers (≥ 8 in `UlipContext`); new methods are new `oneof` variants in `UlipPayload`.
- **For v2**: stand up a separate `v2` service alongside `v1`. Instances negotiate the highest mutually-supported version via `Info`.

Why this matters: an instance running `v1.3` can talk to an instance still on `v1.0` — the older one just ignores unknown fields. No hard cutover.

---

## 10. Recursive envelopes — the clever bit

Look at this payload:

```protobuf
message CreateIncomingRequest {
  string account_did    = 1;
  string token_class_id = 2;
  string amount         = 3;
  google.protobuf.Duration timeout = 4;
  SignedLockProof proof_of_lock = 5;     // ← serialised UlipEnvelope of the source's Lock
}
```

`SignedLockProof` is **bytes containing another full ULIP envelope** — specifically, the **source instance's signed `Lock` envelope**.

What this gives you: when instance B receives `CreateIncoming`, it can verify "yes, instance A actually did lock these funds" by verifying the **nested signed envelope**. The proof travels with the request.

Same trick for `CommitCredit` — it carries a signed `CommitDebit` proof from the source.

This is how the federation gets end-to-end verifiability without a central ledger.

---

## 11. What changes in the existing repos

ULIP itself lives in `units-ulip/`. But each consuming repo has to **add the SDK as a dependency and switch over**:

| Repo                       | Change                                                                                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `units-workflows`          | Add npm deps `@finternet/ulip-canonical`, `@finternet/ulip-client`. Replace the ad-hoc `{context, payload}` envelope in `packages/http-client/src/client.ts` with the real `UlipEnvelope`. |
| `units-api`                | Add Go dep `github.com/finternet/units-ulip-go`. Wrap every Kafka operation in a signed envelope. Server-side handlers built on the SDK's transport package.                               |
| `units-token-runtime`      | Add Rust crates `units-ulip-canonical`, `units-ulip-envelope`. Replace `TokenOperationMessage` with envelope-wrapped form; **engine verifies the envelope before executing**.              |
| `units-registry` (Spec 04) | Depend on `units-ulip-go` for handlers.                                                                                                                                                    |


The conformance test suite **moves out** of the monorepo into `units-ulip/conformance/`.

---

## 12. Tests that must pass (acceptance criteria)

| Test   | What it proves                                                                                               |                                       |
| ------ | ------------------------------------------------------------------------------------------------------------ | ------------------------------------- |
| T-01-1 | Same envelope round-trips through gRPC AND REST; signature verifies on both                                  | Canonical bytes are truly canonical   |
| T-01-2 | Tamper with payload after signing → `UNAUTHENTICATED`                                                        | Signature actually protects integrity |
| T-01-3 | Send same `(txn_id, op_seq)` twice → cached response, no re-execution                                        | Idempotency works                     |
| T-01-4 | Caller not in allowlist → `PEER_DENIED`                                                                      | Allowlist enforced                    |
| T-01-5 | v2 envelope against v1 server → `VERSION_UNSUPPORTED`                                                        | Version negotiation works             |
| T-01-6 | Full transfer flow: `Info → Capabilities → ResolveName → Lock → CreateIncoming → CommitDebit → CommitCredit` | End-to-end happy path                 |
| T-01-7 | `GetAccount` for migrated DID → `FORWARD` with `new_home`                                                    | Migration forwarding works            |
| T-01-8 | Go signer / Rust verifier (and reverse)                                                                      | Cross-language compatibility          |

---

## 13. Cross-spec seams — who depends on ULIP

| Other spec                         | How it uses ULIP                                                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Spec 02 (Token Runtime)**        | Engine verifies the envelope before executing any op. Idempotency cache keyed by `(caller, txn_id, op_seq)`.                   |
| **Spec 03 (Workflows)**            | Every Restate workflow uses `@finternet/ulip-client`, **even for intra-instance calls** (loopback). Keeps the surface uniform. |
| **Spec 04 (Registry)**             | `RegisterName`, `ResolveName`, `Capabilities` are ULIP methods — registry doesn't have its own protocol.                       |
| **Spec 05 (Migration)**            | `InitiateMigration`, `AcceptMigration`, `FinalizeMigration` carried by ULIP.                                                   |
| **Spec 06 (Non-compliance + Obs)** | Lifecycle state surfaces via `Capabilities` response. Archive interceptor stores every envelope.                               |

---

## 14. Explicitly out of scope for v1

- **Delegated capability** semantics — field is reserved (`context.extensions["delegated_capability"]`), but v1 doesn't consume it.
- **Multi-hop routing** (A → B → C) — direct peer-to-peer only.
- **gRPC bidirectional streaming** for token-class subscriptions — `SubscribeTokenClass` is server-streaming only.

---

## 15. The 30-second elevator pitch

> ULIP is the wire language UNITS instances use to talk to each other. Every message is a signed envelope with three parts: context (who/what/when), payload (the actual request), and signature (proof). Both sides agree on EXACT canonical bytes to sign, so a Go signer and a Rust verifier match. Idempotency is `(caller, txn_id, op_seq)` — same key → same cached response, retries are safe. Security is mTLS + envelope signature + replay cache. Lives in its own repo with Go/Rust/TS SDKs so external operators can adopt it without pulling in our monorepo.
