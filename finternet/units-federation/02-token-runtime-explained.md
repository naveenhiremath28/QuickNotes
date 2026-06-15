# Spec 02 — Token Runtime — Plain English Walkthrough

> Source: `units-federation/02-token-runtime.md`
> Owner: Runtime team. Depends on Spec 01 (envelope) + Spec 07 (DB). Consumed by Spec 03 (workflows), Spec 06 (audit anchoring).

---

## 1. The one-line summary

The **Token Runtime** is the Rust engine that **actually mutates token balances**. Spec 02 refactors it so that a cross-instance transfer can **pause halfway**, complete on the other side, or cleanly reverse — instead of being one big destructive operation.

Think of it as **upgrading the bank vault clerk** from "make one big move" to "make a sequence of small, recoverable moves."

---

## 2. The mental model — wire transfer with escrow

Old way (single instance): "subtract 100 from alice, add 100 to bob." One atomic operation. Easy.

New way (across instances): **two banks coordinating**, one operation at a time:

| Step | Bank A (sender) | Bank B (receiver) |
|---|---|---|
| 1 | **Lock** alice's 100 (put a hold, but don't take it yet) | |
| 2 | | **CreateIncoming** — open a slot expecting 100 |
| 3 | **CommitDebit** — now actually subtract 100 | |
| 4 | | **CommitCredit** — now actually add 100 |

Each step is a **separate atomic operation** with its own signed receipt. If step 3 fails, alice's money is still locked (recoverable) — it doesn't vanish.

That's the heart of Spec 02.

---

## 3. The 10 primitives — the new "atomic moves"

The old runtime had **chunky operations** like `Transfer`. The new one breaks them down:

| Group | Primitive | What it does |
|---|---|---|
| **Source-side** (sender's bank) | `Lock` | Reserve funds without spending them |
| | `CommitDebit` | Finalize the subtraction (after destination confirms) |
| | `Unlock` | Release a lock if the transfer is cancelled |
| **Destination-side** (receiver's bank) | `CreateIncoming` | Open a slot expecting incoming funds |
| | `CommitCredit` | Finalize the addition |
| | `RejectIncoming` | Decline the transfer |
| **Supply** (issuer-only) | `Mint` | Create new tokens out of thin air |
| | `Burn` | Destroy tokens |
| **Reversal** | `Credit` | Restore funds (used for reversing a debit) |
| | `Debit` | Take funds (used for reversing a credit) |
| **Legacy** | `Transfer` | Kept as a thin shim — now expressed as `Lock + CreateIncoming + CommitDebit + CommitCredit` inside one txn |

The `Freeze`, `Unfreeze`, `Update` ops are kept unchanged.

---

## 4. The reversal model — the most important idea

**Key principle: reversals are NEW operations, not destructive rewrites.**

If you reverse a debit, you don't go back and delete the debit — you write a **new Credit operation** that has `reverses_txn: <original_txn_id>` in its payload.

Why this matters:
- **Audit trail preserved** — every change is in the chain, forever
- **Tamper detection works** — you can't quietly undo something
- **The commitment chain remains valid** — each op extends it, including reversals

Example flow if a transfer fails halfway:
1. `Lock` (op_seq=0) — alice's 100 reserved ✓
2. `CommitDebit` (op_seq=1) — alice's 100 gone ✓
3. `CreateIncoming` (op_seq=2) — slot opened on B ✓
4. `CommitCredit` (op_seq=3) — **FAILS** ✗
5. `Credit` (op_seq=4, `reverses_txn=this`) — alice restored, with admin authorisation

The original debit is **still in the chain**. The reversal is also in the chain. Both exist.

---

## 5. State model changes — what changes in the token's data

### Before
```rust
pub struct StateData {
    pub status: TokenStatus,
    pub supply: Option<Supply>,
    pub lock_state: Option<LockState>,    // ← single lock only
    // ...
}
```

### After
```rust
pub struct StateData {
    pub status: TokenStatus,
    pub supply: Option<Supply>,
    pub locks: Vec<LockState>,            // ← multiple locks (one per (txn_id, op_seq))
    pub incoming: Vec<IncomingState>,     // ← NEW: pending incoming transfers
    // ...
}
```

### Why a vector of locks?

Previously a token could be locked exactly once. Now multiple workflows can lock different portions for different `(txn_id, op_seq)` pairs. Each lock has its own:
- `txn_id` — which workflow owns it
- `op_seq` — which step
- `amount` — how much is locked (partial locks supported)
- `locked_until` — expiry

### What is `IncomingState`?

A new bucket per token tracking "transfers I'm expecting":

```rust
pub struct IncomingState {
    pub txn_id:        String,
    pub op_seq:        u32,
    pub source_inst:   String,      // who's sending
    pub amount:        String,
    pub created_at:    DateTime,
    pub expires_at:    DateTime,
    pub proof_of_lock: SignedEnvelopeBytes,  // ← the source's signed Lock envelope
}
```

The `proof_of_lock` is the **source's signed Lock envelope embedded inside** — so the destination can verify "yes, the source really did lock the funds." (This is the recursive-envelope trick from Spec 01.)

### Three derived balance views per account+class

Computed on every primitive call, written to `account_balance_views` (a Postgres materialized view):

- `available` — unlocked, owned balance
- `locked_total` — sum of all `LockState.amount`
- `incoming_total` — sum of all `IncomingState.amount`

---

## 6. Inside one primitive — what actually happens (Lock example)

```rust
pub async fn execute_lock(ctx, op, mut state) -> ProgramResult<OperationResult> {
    // 1. Validate
    require_unfrozen(&state)?;
    require_class_match(&state, &p.token_class_id)?;
    require_balance_at_least(&state, &p.amount)?;

    // 2. Mutate state
    state.state.locks.push(LockState {
        txn_id:       Some(ctx.tx_id),
        op_seq:       Some(ctx.op_seq),
        amount:       Some(p.amount),
        locked_until: Some(ctx.timestamp + p.timeout),
        // ...
    });
    _move_to_locked(&mut state, &p.amount)?;
    _recompute_balances(&mut state);

    // 3. Extend the commitment chain
    program.compute_state_commitment(&state);

    // 4. Emit signed proof
    Ok(OperationResult::ok(state).with_proof(emit_lock_proof(ctx, p)))
}
```

The pattern is the same for every primitive: **validate → mutate → recompute → re-hash commitment → emit signed proof**.

---

## 7. Internal helpers — building blocks NOT exposed via `OperationPayload`

The runtime ships a set of underscored helpers that **programs use directly**, never reachable from the wire:

| Helper | What |
|---|---|
| `_credit_balance(state, amount)` | Add to available balance |
| `_debit_balance(state, amount)` | Subtract from available balance |
| `_move_to_locked(state, amount)` | Available → locked |
| `_drain_locked(state, amount)` | Locked → gone |
| `_reserve_incoming(state, slot)` | Add an IncomingState entry |
| `_consume_incoming(state, txn_id, op_seq)` | Remove an IncomingState, return amount |
| `_update_supply(state, delta)` | Adjust total supply |
| `_recompute_balances(state)` | Recalculate available / locked_total / incoming_total |
| `_validate_authorisation(op, auth)` | Check issuer/admin auth claim |
| `_validate_jurisdictional_policy(class_id, ctx)` | Check token class hosting rules |

These are **invariant-preserving** — programs compose them instead of mutating fields directly. Prevents accidental "subtract from locked without crediting back" bugs.

---

## 8. Commitment chain — every op extends it

The existing chain already hashes `previous_commitment + last_tx_id + state + state_version`. Spec 02 adds:

- Use the **same canonical serializer** as Spec 01 (`units-ulip-canonical` crate from the ULIP repo)
- Hash inputs **explicitly include** `op_seq` and `reverses_txn` (when present)
- Each op produces its **own commitment node** — even within one txn_id, op_seq=0/1/2/3 each extend the chain
- The chain is **per-account**, not per-txn

```rust
pub fn compute_state_commitment_with_op(state, op, ctx, config) -> String {
    let bytes = units_ulip_canonical::canonicalize(&CommitmentInput {
        previous_commitment, txn_id, op_seq, op_type,
        account_did, token_class_id, amount,
        reverses_txn,        // ← critical: reversals are linked
        timestamp_ms, state_version,
    });
    sha256_or_blake3(&bytes)
}
```

The "**signed tip**" of each account's chain is published periodically by the engine — Spec 06 handles that.

---

## 9. The engine pipeline — what happens for every Kafka message

```
1. Pull envelope from Kafka
2. Verify envelope (Spec 01) — caller signature, version, peer allowlist
3. Look up idempotency cache by (caller_instance, txn_id, op_seq)
   - If AlreadyProcessed: return cached response, commit Kafka offset, STOP
4. Begin DB tx
5. Load token state, token class config
6. Run pre-hooks: commitment verification, jurisdictional check
7. Dispatch to TokenProgram::execute(ctx, op, state)
8. Persist: new state, commit_state record, token_transaction record
9. Update transaction status
10. Store response in idempotency cache
11. Commit DB tx
12. Emit downstream events (Kafka, observability stream)
13. Commit Kafka offset
```

**Steps 2, 3, 10 are the federation-aware additions.** The rest is the existing flow.

---

## 10. Idempotency cache — the Postgres table that makes retries safe

```rust
pub struct IdempotencyCache { /* Postgres-backed */ }

impl IdempotencyCache {
    pub async fn check_or_insert(&self, caller, txn_id, op_seq) 
        -> Result<IdempotencyOutcome>;
    pub async fn store_response(&self, caller, txn_id, op_seq, response) 
        -> Result<()>;
}

pub enum IdempotencyOutcome {
    NewRequest,
    AlreadyProcessed { cached_response: Vec<u8> },
}
```

Table: `ulip_idempotency_log (caller_instance, txn_id, op_seq, response_bytes, expires_at)`. Same `(caller, txn_id, op_seq)` → return cached response, do NOT re-execute the primitive.

---

## 11. Transaction lifecycle — new states

The old runtime had: `Submitted, Pending, Processing, Completed, Failed, Cancelled, AwaitingSignature`.

New federation-aware states added:

| State | Meaning |
|---|---|
| `Prepared` | Lock placed, waiting for destination |
| `Committing` | Commit phase in progress |
| `AutoReversing` | Destination didn't respond — auto-reversal triggered |
| `Reversed` | Reversal complete, balance restored |
| `Stuck` | Destination unreachable; needs operator intervention |

These match the workflow state machine in Spec 03.

---

## 12. Token Manager — a brand-new service

**The problem this solves**: today's `alchemyAdapter` publishes `finternet.transactions.{submit,confirm,status,dlq}` events when on-chain transactions happen. **Nothing consumes the `status` events** to write proxy ledger entries. So there's no record on the UNITS side that "this account holds 10 USDC on Ethereum."

### The new service

`units-services/tokenManager/` (Go, new):

```
tokenManager/
├── consumers/
│   └── transactions_status.go      # consumes finternet.transactions.status
├── ledger/
│   ├── proxy_view.go               # writes ledger entries
│   └── repository.go
├── ulip/
│   ├── client.go                   # outbound RecordProxyEntry
│   └── server.go                   # inbound RecordProxyEntry
```

### What it does

For each `transactions.status = confirmed` event:

1. Identify affected on-chain addresses (sender + recipient)
2. Resolve each on-chain address → UNITS account DID (via units-api)
3. Determine each account's **home instance**
4. Emit a **`RecordProxyEntry` ULIP request** to each home instance, signed by the local instance
5. The receiving ULIP server writes a ledger view entry under that account's token state

### `RecordProxyEntry` payload

```protobuf
message RecordProxyEntryRequest {
  string account_did      = 1;
  string token_class_id   = 2;
  string delta            = 3;     // signed amount, decimal string
  string tx_hash          = 4;
  string on_chain_chain   = 5;
  uint64 block_number     = 6;
  string operator_signing_instance = 7;
}
```

**Authorisation rule**: the envelope must be signed by an instance registered as a **Token Manager for that chain** in the recipient's `Capabilities` document (Spec 04). Not any old instance can record proxy entries.

---

## 13. Migration of existing data (Spec 07 owns the full plan)

Runtime-side requirements:
1. Existing `lock_state: Option<LockState>` → `locks: Vec<LockState>` with one entry
2. New `incoming: Vec<IncomingState>` initialised empty
3. Existing completed transactions get a synthetic `op_seq = 0` for backwards compat
4. Legacy `Transfer` op continues to work — now expressed as a four-op atomic burst inside one `txn_id`

---

## 14. What files change

### New files

| Path | Purpose |
|---|---|
| `tokenPrograms/programs/reference-ft/src/operations/{commit_debit,create_incoming,commit_credit,reject_incoming,credit,debit,record_proxy_entry}.rs` | New primitives |
| `tokenPrograms/programs/core/src/helpers.rs` | Internal building blocks |
| `tokenPrograms/programs/core/src/auth.rs` | `_validate_authorisation` |
| `tokenEngine/src/engine/idempotency.rs` | (txn_id, op_seq) cache |
| `tokenEngine/src/engine/hooks/envelope_verification.rs` | Verify envelope before execute |
| `units-services/tokenManager/` | Whole new service |

### Modified files

| File | Change |
|---|---|
| `tokenPrograms/interface/src/types.rs` | New `OperationPayload` variants, `LockState` extension, `IncomingState`, `StateData.locks`/`incoming` |
| `tokenPrograms/programs/reference-ft/src/program.rs` | Dispatch new variants |
| `tokenPrograms/programs/core/src/commitment.rs` | Use ULIP canonical serializer; explicit `op_seq` + `reverses_txn` |
| `tokenEngine/src/engine/executor.rs` | New 13-step flow |
| `tokenEngine/src/kafka/consumer.rs` | Consume envelope-wrapped messages |
| `tokenEngine/src/kafka/message.rs` | Replace `TokenOperationMessage` with `UlipEnvelope` parsing |
| `tokenEngine/src/db/models/transaction.rs` | New lifecycle enum variants |
| `tokenEngine/src/db/repositories/*.rs` | Persist new state shape, new statuses |

---

## 15. Tests that must pass

| Test | What it proves |
|---|---|
| T-02-1 | `Lock → CommitDebit` happy path: available → locked → gone; commitment extends |
| T-02-2 | `Lock → Unlock`: available → locked → available; commitment extends |
| T-02-3 | `CreateIncoming → CommitCredit`: ∅ → incoming → available |
| T-02-4 | `CreateIncoming → RejectIncoming`: ∅ → incoming → ∅ |
| T-02-5 | `Lock → CommitDebit` after lock expires → `LOCK_TIMEOUT_EXPIRED` |
| T-02-6 | Replay same `(txn_id, op_seq)` → cached response |
| T-02-7 | `Credit` with `reverses_txn` + valid auth → balance restored, chain references reversal |
| T-02-8 | `Debit` reversal where balance insufficient → `INSUFFICIENT_BALANCE`, no state change |
| T-02-9 | Two concurrent `Lock`s on same token → both succeed, `locked_total` sums |
| T-02-10 | Invalid envelope signature → `UNAUTHENTICATED`, no state change |
| T-02-11 | Legacy `Transfer` op still works (backwards compat) |
| T-02-12 | Token Manager records proxy entry → recipient instance shows updated proxy-class balance |

---

## 16. Cross-spec seams

| Other spec | How it touches Spec 02 |
|---|---|
| **Spec 01 (ULIP)** | Engine consumes ULIP envelope; signature verifier is the ULIP crate |
| **Spec 07 (DB)** | Owns the migrations for `state.locks`, `state.incoming`, lifecycle enum, idempotency table |
| **Spec 03 (Workflows)** | Workflows compose these primitives — each step is **one primitive identified by `(txn_id, op_seq)`** |
| **Spec 04 (Registry)** | `_validate_jurisdictional_policy` reads from local capability directory |
| **Spec 06 (Observability)** | Engine signs the per-account commitment tip and emits to observability stream |
| **Spec 03 (Swap)** | Swap workflow waits on `transactions.status`, then invokes Token Manager for proxy bookkeeping |

---

## 17. Out of scope

- **Multi-currency atomic swap** inside the runtime (composing two locks + two incomings). That's workflow-level — runtime stays atomic-per-op.
- **Partial-fill orders.** `amount` is exact; partial settlement is workflow-level.
- **Cross-program operations** (e.g., NFT-fungible swap). Runtime hosts independent programs.

---

## 18. 30-second elevator pitch

> Spec 02 refactors the Rust Token Runtime so a cross-instance transfer can pause halfway. The chunky `Transfer` op is replaced by 10 atomic primitives — `Lock`, `CommitDebit`, `Unlock` on the source; `CreateIncoming`, `CommitCredit`, `RejectIncoming` on the destination; `Credit`/`Debit` for reversals; `Mint`/`Burn` for supply. Each primitive is keyed by `(txn_id, op_seq)`, signed via ULIP envelopes, idempotency-cached in Postgres, and extends a per-account SHA-256/Blake3 commitment chain. Reversals are NEW ops that point at `reverses_txn` — no destructive rewrites. A brand-new **Token Manager** service consumes `finternet.transactions.status` Kafka events from the chain adapter and mirrors on-chain transfers into UNITS proxy ledger views via the `RecordProxyEntry` ULIP method.
