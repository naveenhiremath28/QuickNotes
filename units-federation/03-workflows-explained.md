# Spec 03 — Workflows — Plain English Walkthrough

> Source: `units-federation/03-workflows.md`
> Owner: Workflow team. Depends on Spec 01 (envelope), 02 (primitives), 04 (registry), 07 (DB). Consumed by Spec 05 (Migration is a workflow), Spec 08 (E2E tests).

---

## 1. The one-line summary

If **Spec 02** gave us atomic primitives like `Lock`, `CommitDebit`, `CreateIncoming`, `CommitCredit` — **Spec 03 builds the orchestrators that call them in the right order**, handle failures, retry intelligently, and survive process crashes.

Workflows = **the conductor**. Primitives = **the instruments**. ULIP = **the language they speak**.

---

## 2. The mental model — a flight attendant's safety checklist

A cross-instance transfer isn't one step — it's a **scripted sequence** with strict ordering:

1. Confirm destination exists (resolve name)
2. Place hold on source
3. Open slot on destination
4. Commit debit on source
5. Commit credit on destination
6. Mark transaction complete

If step 4 fails, you don't say "oh well." You **roll back step 2** (release the lock). If step 5 fails after step 4 succeeded, you issue a **reversing credit** (because the debit can't be un-done — Spec 02's reversal model).

The workflow is the script. **Restate** is the durable journal that ensures the script restarts where it stopped if the process crashes.

---

## 3. The four workflows

| Workflow | Purpose | Path |
|---|---|---|
| **Signup** | New user joins the federation: claims a name, gets a DID, gets an account | `apps/signup/` |
| **Transfer** | Move tokens A → B (cross-instance or loopback same-instance) | `apps/transfer/` |
| **Swap** | Bridge: native ↔ on-chain proxy (e.g., NFH-T ↔ USDC on Ethereum) | `apps/swap/` |
| **Migration** | Move an account from one instance to another | `apps/migration/` (Spec 05 fills in semantics) |

All four are **new Restate services**. They live alongside the two existing ones (`profile-update`, `delegation`).

---

## 4. Why Restate? — durable workflows in plain terms

A workflow is a long-running script that **survives process crashes**. Restate gives us:

- **Durable journal** — every step is recorded; if the process dies mid-step, it restarts from where it stopped, NOT from the beginning.
- **`ctx.run(name, fn)`** — wraps any function so it runs **at-most-once** under replay. Run it twice in code? Second call returns the cached first result.
- **Workflow ID** — every workflow instance has a unique ID. We use it as the `txn_id` for all the ULIP envelopes it emits.

This is **critical** for federation. Without it: a crash mid-transfer = your lock is stuck forever and nobody knows how to clean it up. With it: the workflow resumes and finishes the dance.

---

## 5. The "no intra-instance fast path" rule

The architecture **forbids** a shortcut for same-instance transfers. Even if alice and bob are on the **same instance**, the workflow still:

- Wraps every primitive in a ULIP envelope
- Signs it
- Sends it through the local ULIP server (loopback)
- Verifies the signature on the receiving side
- Hits the idempotency cache

Why? **One code path for everything**. No "if same-instance → bypass, else → remote" branches. Easier to reason about, easier to test, no risk of the fast path missing a security check.

---

## 6. Shared utilities — the toolkit every workflow uses

A new package `packages/workflow-utils/` ships these helpers:

### `op-seq.ts` — monotonic counter
```typescript
export interface OpSeqAllocator {
  next(): number;     // 0, 1, 2, ... persisted to Restate state
}
```
Persisted to `ctx.set("op_seq_cursor", n)` so replays produce the same sequence — same `(txn_id, op_seq)` for every step on every replay.

### `ulip-call.ts` — the wrapper around every primitive
```typescript
export async function ulipCall(ctx, client, method, payload, opSeq): Promise<UlipResponse>
```
Wraps payload in an envelope, signs it, sends it, awaits the response, verifies the response signature. **Idempotent under Restate replay** because `(txn_id, op_seq)` is deterministic.

### `registry-resolve.ts` — name → instance lookup
```typescript
export async function resolveName(ctx, client, name): Promise<RegistryRecord>
```
Queries Spec 04's registry. Returns home instance, DID, lifecycle state.

### `lifecycle.ts` — the state machine vocabulary
```typescript
export type Lifecycle =
  | "SUBMITTED" | "PREPARED" | "COMMITTING" | "COMMITTED"
  | "ABORTED" | "AUTO_REVERSING" | "REVERSED" | "STUCK";

export async function setStatus(ctx, repo, txnId, status): Promise<void>;
```

### `retry-policy.ts` — the retry behaviour
```typescript
export const defaultRetryPolicy = {
  maxDurationMs:   48 * 60 * 60 * 1000,   // 48 hours
  initialBackoffMs: 60_000,                // 1 min
  maxBackoffMs:    60 * 60 * 1000,         // 1 hour
  maxAttempts:     15,
};
```

---

## 7. Signup workflow — the simplest one

```typescript
export const signupWorkflow = restate.workflow({
  name: "signup",
  handlers: {
    run: async (ctx, req: SignupRequest) => {
      const txnId = ctx.workflowId();
      await setStatus(ctx, repo, txnId, "SUBMITTED");

      // 1. Find an instance that accepts new users
      const instances = await ctx.run("list-instances", () =>
        ulipCall(ctx, client, "Capabilities", { acceptsNew: true }, 0));
      const home = req.preferredHome ?? defaultHome(instances);

      // 2. Generate DID locally (deterministic — auth proof + workflowId)
      const did = await ctx.run("gen-did", () =>
        generateDid(req.authProof, txnId));

      // 3. Register the name globally
      const reg = await ctx.run("register-name", () =>
        ulipCall(ctx, client, "RegisterName", { name: req.name, did, home }, 0));

      // 4. Create the local account
      await ctx.run("create-account", () =>
        unitsApi.createAccount({ did, name: req.name, home }));

      await setStatus(ctx, repo, txnId, "COMMITTED");
      return { name: req.name, did };
    },
  },
});
```

### Failure handling
- `RegisterName` returns `NAME_TAKEN` → `TerminalError`, status `ABORTED`. End of story.
- `RegisterName` succeeds but local account creation fails → Restate retries `create-account`. Idempotent: re-running with same DID is a no-op.

### Why use a workflow for signup at all?
Signup isn't really "durable" in the federation sense — it's one round-trip. But modelling it as a workflow gives one place to **retry the global step** (`RegisterName`) if local creation fails. Centralised retry logic > scattered try/catch.

---

## 8. Transfer workflow — the four-step dance

This is the workflow you'll see in your head most often.

```typescript
export const transferWorkflow = restate.workflow({
  name: "transfer",
  handlers: {
    run: async (ctx, req: TransferRequest) => {
      const txnId = ctx.workflowId();
      const opSeq = makeOpSeqAllocator(ctx);
      await setStatus(ctx, repo, txnId, "SUBMITTED");

      // 1. Resolve recipient via registry
      const recipient = await ctx.run("resolve", () =>
        resolveName(ctx, client, req.toName));

      // 2. PREPARE — reversible operations
      await setStatus(ctx, repo, txnId, "PREPARED-WIP");
      const lockProof = await ctx.run("lock", () =>
        ulipCall(ctx, client, "Lock", { ... }, opSeq.next()));

      const incomingProof = await ctx.run("create-incoming", () =>
        ulipCall(ctx, client, "CreateIncoming", {
          ...,
          proofOfLock: lockProof.envelopeBytes,   // ← recursive envelope
        }, opSeq.next()));
      await setStatus(ctx, repo, txnId, "PREPARED");

      // 3. COMMIT — point of no return; use retry wrapper
      await setStatus(ctx, repo, txnId, "COMMITTING");
      const commitProof = await commitWithRetry(ctx, () =>
        ulipCall(ctx, client, "CommitDebit", { ... }, opSeq.next()));

      await commitWithRetry(ctx, () =>
        ulipCall(ctx, client, "CommitCredit", {
          ...,
          commitProof: commitProof.envelopeBytes,   // ← recursive again
        }, opSeq.next()));

      await setStatus(ctx, repo, txnId, "COMMITTED");
      return { txnId, status: "COMMITTED" };
    },
  },
});
```

### The state-machine in plain English

```
SUBMITTED       — workflow just started
   ↓
PREPARED-WIP    — locks being placed
   ↓
PREPARED        — both sides reserved; reversible
   ↓
COMMITTING      — finalizing; point of no return
   ↓
COMMITTED       — done

Off-paths:
ABORTED         — workflow gave up before commit (clean)
AUTO_REVERSING  — commit failed; running the reversal
REVERSED        — reversal complete, balance restored
STUCK           — can't determine destination status, needs human
```

---

## 9. `commitWithRetry` — the magic that handles destination failure

This is the **single most important piece of logic** in the spec.

```typescript
async function commitWithRetry(ctx, fn) {
  return withRetryPolicy(ctx, fn, defaultRetryPolicy, async () => {
    // Reached max retries. Time to figure out what state we're in.
    await setStatus(ctx, repo, ctx.workflowId(), "AUTO_REVERSING");

    const status = await ctx.run("dest-status", () => queryDestinationStatus());

    if (status === "NOT_COMMITTED") {
      // Destination definitely didn't commit. Safe to reverse our debit.
      await ctx.run("reverse-debit", () =>
        ulipCall(ctx, client, "Credit", {
          reversesTxn: ctx.workflowId(),
          authorisation: protocolReversalAuth(),
        }, opSeq.next()));
      await setStatus(ctx, repo, ctx.workflowId(), "REVERSED");
      throw new restate.TerminalError("AUTO_REVERSED");
    }

    if (status === "COMMITTED") {
      // Surprise — it actually went through; we just didn't get the response.
      await setStatus(ctx, repo, ctx.workflowId(), "COMMITTED");
      return /* synthesise success */;
    }

    // status === "UNKNOWN" — we can't tell. Don't auto-reverse blindly.
    await setStatus(ctx, repo, ctx.workflowId(), "STUCK");
    throw new restate.TerminalError("STUCK");
  });
}
```

### Why this matters
The classic distributed-systems trap: "the destination didn't respond — should I reverse?" Wrong! It might have committed and the response just got lost. **You must ask first.** That's `queryDestinationStatus()` — a read-only check to see what actually happened on the other side.

Three outcomes:
- **NOT_COMMITTED** → safe to reverse
- **COMMITTED** → it happened, mark success
- **UNKNOWN** → can't decide, mark `STUCK`, operator handles it manually

---

## 10. Swap workflow — the tricky one (because on-chain is irreversible)

The architecture mandates a specific ordering rule:

> **Reserve (reversible) → On-chain (irreversible) → Local finalize (reversible)**
>
> On-chain step is **never at the end**.

Why? If on-chain were last and it failed, you'd have native tokens credited and nothing on-chain — money lost. By putting on-chain in the middle, you can:
- Reverse the reservation if on-chain fails (pre-confirmation)
- Retry the local finalize until it succeeds (post-confirmation)
- If local finalize **truly** can't succeed → run **compensation** (send the on-chain tokens back to the user)

### swap_proxy_to_native (alice pays USDC, receives NFH-T)

```typescript
// 1. Reserve native (reversible)
await ctx.run("reserve-native", () =>
  ulipCall(ctx, client, "CreateIncoming", { ..., timeout: 600s }, opSeq.next()));

// 2. On-chain (IRREVERSIBLE)
const txHash = await ctx.run("proxy-submit", () =>
  adapter.proxySubmit({ chainId, fromAddr, toAddr: operatorTreasury, ... }));

const confirmation = await ctx.run("await-confirmation", () =>
  adapter.awaitConfirmation(chainId, txHash, { maxWaitMs: 600_000 }));

if (!confirmation.confirmed) {
  // Pre-confirmation failure — clean reject, no gas wasted
  await ctx.run("reject-incoming", () =>
    ulipCall(ctx, client, "RejectIncoming", { ... }, opSeq.next()));
  await setStatus(ctx, repo, txnId, "ABORTED");
  return { status: "ABORTED" };
}

// 3. Finalize (local, retry until success)
await commitWithRetry(ctx, () =>
  ulipCall(ctx, client, "CommitCredit", { ... }, opSeq.next()));

await ctx.run("record-proxy-debit", () =>
  ulipCall(ctx, client, "RecordProxyEntry", {
    delta: `-${proxyAmount}`,
    txHash, onChainChain,
  }, opSeq.next()));
```

### Compensation path (post-confirmation failure)

If on-chain succeeded but local commit still can't succeed after retries:

1. Workflow enters `STUCK`
2. Operator queue notified (Spec 06)
3. After SLA (default 30 min) → **auto-compensation task** submits an on-chain transfer back to the user
4. Local record updated; audit entry emitted
5. `transactions.auto_reversal_reason = "post-confirm commit failure"`

The user gets their on-chain tokens back. The operator absorbs the gas cost. Not free for the operator — but the user's never holds half-completed state.

---

## 11. Migration workflow — just a shell here

Spec 05 owns the migration algorithm. Spec 03 only wires the Restate workflow shell:

```typescript
export const migrationWorkflow = restate.workflow({
  name: "migration",
  handlers: {
    run: async (ctx, req: MigrationRequest) => {
      // 1. Initiate (signed envelope from source)
      // 2. Accept (signed envelope from destination)
      // 3. Freeze + Ship (atomic on source)
      // 4. Install + sign (on destination)
      // 5. Update registry
      // 6. Install forwarding pointer (grace period)
      // ← Spec 05 fills these in
    },
  },
});
```

---

## 12. Restate persistence + idempotency

The four rules that keep workflows idempotent under replay:

1. **`txn_id == workflowId()`** — every primitive call from the same workflow carries the same `txn_id`.
2. **`op_seq` from persisted counter** — `ctx.set("op_seq_cursor", n)` survives replays, so step 3 of the workflow always emits the same `(txn_id, op_seq=3)`.
3. **Every primitive wrapped in `ctx.run(name, fn)`** — Restate caches the result; replays return the cached result without re-running.
4. **Spec 02's idempotency cache** — even if Restate's cache is empty, the receiving instance's `(caller, txn_id, op_seq)` cache returns the cached response.

Two layers of idempotency stacked = retries are **always** safe.

---

## 13. Workflow record persistence

Two Prisma tables (Spec 07 owns migrations):

```prisma
model Workflow {
  txnId            String   @id @map("txn_id")          // ← renamed from workflowId
  workflowName     String
  callerInstance   String?
  initiatorAccount String?
  status           String   @default("SUBMITTED")
  payload          Json
  outcome          Json?
  retryAttempts    Int      @default(0)
  lastError        String?
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
}

model WorkflowOp {
  txnId   String
  opSeq   Int
  opType  String
  details Json
  result  Json?
  @@id([txnId, opSeq])
}
```

`Workflow` = the parent transaction. `WorkflowOp` = each step. Together they mirror Restate's durable journal in Postgres for query-friendly access.

---

## 14. Two deployment modes

| Mode | When | Path |
|---|---|---|
| **Gateway mode** | Small / dev — single Restate service binds all four new workflows + the two existing ones | `apps/gateway/src/app.ts` |
| **Per-app mode** | Production — `signup`, `transfer`, `swap`, `migration` each run as their own Restate service | One pod each |

Gateway mode is for convenience. Per-app mode lets you scale each workflow independently.

---

## 15. Configuration knobs

| Env var | Default | Purpose |
|---|---|---|
| `ULIP_BASE_URL` | — | Local ULIP server URL (loopback default) |
| `ULIP_SIGNING_KEY_REF` | — | Vault path for service's signing key |
| `REGISTRY_DNS_DOMAIN` | — | DNS domain hosting the registry |
| `RETRY_MAX_DURATION_MS` | 48 h | Retry deadline |
| `RETRY_INITIAL_BACKOFF_MS` | 1 min | First backoff |
| `RETRY_MAX_BACKOFF_MS` | 1 h | Backoff ceiling |
| `RETRY_MAX_ATTEMPTS` | 15 | Hard cap |
| `SWAP_AUTO_COMPENSATION_DELAY_MS` | 30 min | SLA before auto-compensating a stuck swap |
| `SWAP_LOCK_TIMEOUT_MS` | 10 min | Native lock duration on swap |

---

## 16. File changes

### New
```
units-workflows/apps/{signup,transfer,swap,migration}/
├── package.json
├── src/{app.ts,register.ts,schemas.ts,workflow.ts,clients/}
└── tests/

units-workflows/packages/workflow-utils/
└── src/{op-seq,ulip-call,registry-resolve,lifecycle,retry-policy,types}.ts
```

### Modified
| File | Change |
|---|---|
| `apps/gateway/src/app.ts` | Bind the four new workflows |
| `packages/http-client/src/client.ts` | Replace ad-hoc envelope with `@finternet/ulip-client` |
| `packages/workflow-db/prisma/schema.prisma` | Apply new schema |
| `units-api/src/routers/workflow.go` | Route the four new workflow names to Restate |
| `units-api/specs/db/workflow_registry.sql` | Seed rows for the four workflows |

---

## 17. Tests that must pass

| Test | What it proves |
|---|---|
| T-03-1 | Signup happy path: DID generated, name registered, account created |
| T-03-2 | Signup with `NAME_TAKEN`: status `ABORTED` |
| T-03-3 | Transfer same-instance: all 4 primitives via loopback, commitments updated |
| T-03-4 | Transfer cross-instance: 2 primitives remote, commitments linked by `txn_id` |
| T-03-5 | Transfer with unreachable destination during commit: retries → `AUTO_REVERSING` → `Credit` reversal → `REVERSED` |
| T-03-6 | Transfer where status query also fails: `STUCK`, operator notified |
| T-03-7 | Swap proxy → native happy path: NFH-T credited, USDC ledger debited, txHash recorded |
| T-03-8 | Swap with on-chain timeout pre-confirmation: `RejectIncoming`, `ABORTED`, no gas spent |
| T-03-9 | Swap with on-chain success but local commit failure: `STUCK` → auto-compensation after SLA |
| T-03-10 | Restate replay of mid-flight transfer: same `op_seq`, cached responses returned |
| T-03-11 | Workflow record persisted to `workflows` + `workflow_ops` matching Restate journal |

---

## 18. Cross-spec seams

| Other spec | How it touches Spec 03 |
|---|---|
| **Spec 01 (ULIP)** | `@finternet/ulip-client` is the package wrapping every primitive call |
| **Spec 02 (Runtime)** | Workflows compose the primitives the runtime exposes |
| **Spec 04 (Registry)** | `RegisterName`, `ResolveName`, `Capabilities` are called via ULIP |
| **Spec 05 (Migration)** | Migration workflow shell here, semantics there |
| **Spec 07 (DB)** | New schema for `workflows` + `workflow_ops` |
| **Spec 06 (Observability)** | Detects `STUCK`, manages auto-compensation SLA |

---

## 19. Out of scope

- **Concurrent multi-currency transfers** (multi-asset atomic swaps in one workflow). v1 = single asset path per workflow.
- **Gas fee abstraction** (operator subsidies). Tracked separately under proxy-token ops.
- **Cross-instance multi-hop routing** (A → B → C). Exactly one source, one destination per transfer.

---

## 20. 30-second elevator pitch

> Spec 03 implements four Restate workflows — Signup, Transfer, Swap, Migration — that orchestrate Spec 02's atomic primitives via ULIP envelopes (Spec 01). Every workflow uses `txn_id = workflowId()` and a monotonic `op_seq` so retries are idempotent across two layers (Restate's journal + the runtime's primitive cache). Transfer follows `PREPARED → COMMITTING → COMMITTED`; if commit fails, `commitWithRetry` queries the destination's actual status — `NOT_COMMITTED` triggers a `Credit` reversal, `UNKNOWN` enters `STUCK`. Swap enforces ordering `Reserve → On-chain → Finalize` so the irreversible on-chain step is never last; post-confirmation failure triggers operator compensation after an SLA. Even same-instance transfers go through the ULIP loopback — no fast path, one code path for everything.
