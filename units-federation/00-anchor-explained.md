
```bash
Resume this session with:
claude --resume 9df57e15-dba3-498e-a370-5632e5990af6
```
# UNITS Federation — Plain English Walkthrough

> Source: `units-federation/00-anchor.md` (anchor doc dated 2026-04-28, last impl update 2026-05-03)
> This file is a "translate-to-simple-terms" companion. The anchor itself is the source of truth.

---

## 1. The one-line summary

**Today** UNITS is a single bank-like server running by itself.
**After this work** UNITS becomes a "member of a network" — many independent UNITS servers (each fully sovereign, each owning its users and ledger) that talk to each other over a shared protocol so a user on instance A can send tokens to a user on instance B without either side trusting a central operator.

Think: from a single bank → SWIFT-like federation, but cryptographically verifiable end-to-end.

---

## 2. The mental model (the analogy that makes everything click)

Picture **email**:

| Email concept | UNITS Federation equivalent |
|---|---|
| `alice@gmail.com` | `alice@a.finternet` — name + instance |
| Gmail server | A UNITS instance (one company / regulator's deployment) |
| SMTP envelope | **ULIP** envelope (signed, versioned, ID'd) |
| DNS MX record | UNITS **Global Registry** (DNS-backed) |
| Receiving a mail with attachment | Receiving a token transfer |
| Spam reputation / blocklist | **Non-compliance lifecycle** (Watch / Restricted / Quarantined / De-listed) |

Critical difference vs email:
- Every message is **cryptographically signed** by both the user (their wallet key) AND the sending instance.
- Both ledgers must agree on the outcome of every transfer (debit on A, credit on B), so the protocol is "two-phase" instead of fire-and-forget.

---

## 3. What's actually being built (the 8 deliverables)

The work is split into 8 "sub-specs". Each one is a workstream a separate team can own.

### Spec 01 — ULIP Protocol (the language)
ULIP = **Unified Interledger Protocol**. The signed envelope format + the catalog of "methods" instances can call on each other (`Lock`, `CreateIncoming`, `RegisterName`, `ResolveName`, `InitiateMigration`, etc.). Published as its **own repo** (`units-ulip/`) with Go/Rust/TS SDKs so nothing in the monorepo reaches into the protocol's internals.

### Spec 02 — Token Runtime refactor (the atomic moves)
Today the Rust token engine has chunky operations like `Transfer`. Federation needs to split these into **smaller atomic primitives** so a transfer can pause halfway:
- `lock`, `commit_debit`, `unlock` — what the **sender's** instance does
- `create_incoming`, `commit_credit`, `reject_incoming` — what the **receiver's** instance does
- `credit`, `debit`, `mint`, `burn` — building blocks
- **Reversals are new ops** (a "reverse" debit is just another debit pointing back at the original `reverses_txn`) — no destructive rewrites.

### Spec 03 — Workflows (the orchestrators)
Restate-based durable workflows that string the primitives together: `Signup`, `Transfer`, `Swap`, `Migration`. They handle retries, timeouts, and auto-reversal when the other side never responds.

### Spec 04 — Global Registry & Discovery (the phone book)
DNS-backed (yes, real DNS records) registry that maps:
- Instance name → endpoint URL
- Instance name → capabilities (which token classes, which workflows, which versions of ULIP it speaks)
- Instance name → lifecycle state (healthy? quarantined? de-listed?)

Two ULIP methods: `RegisterName` (claim a name), `ResolveName` (look one up), `Capabilities` (fetch the signed capability artefact).

### Spec 05 — Account Migration (porting your account)
"I'm leaving instance A, moving to instance B, but keep my name + DID + balance." Cooperative protocol: `InitiateMigration` → `AcceptMigration` → `FinalizeMigration`. After move, A keeps a **forwarding pointer** for a grace period (like mail forwarding).

### Spec 06 — Non-Compliance + Observability (the regulator's view)
- Instances **archive signed envelopes** of everything they did.
- They **push metrics** to the global telemetry layer.
- An instance's lifecycle can degrade: `watch` → `restricted` → `quarantined` → `de-listed` → `wound-down`.
- **Forced reconstruction**: if an instance is de-listed, governance can force-rebuild affected accounts on a healthy peer using the signed envelope archive.

### Spec 07 — Data Model & Migration Plan
The Postgres / YugabyteDB schema changes + scripts to migrate from today's single-instance shape (e.g., adding `instance_namespace` columns, new lock tables, etc.). DDL phasing matters — schema is the contract.

### Spec 08 — Test Strategy & Rollout
Two-instance docker-compose harness, chaos tests, multi-laptop dev environment, CI matrix.

---

## 4. Dependency order — think of it as **floors of a building**

**The core idea**: some things must exist before other things can sit on top — exactly like you need walls before a roof. Read this bottom-up.

```
┌─────────────────────────────────────────────────────────┐
│ FLOOR 5 (the roof)                                       │
│   08 Tests — only meaningful once everything else exists │
├─────────────────────────────────────────────────────────┤
│ FLOOR 4 (features)                                       │
│   05 Migration   (a feature that rides on workflows)     │
├─────────────────────────────────────────────────────────┤
│ FLOOR 3 (the orchestrator)                               │
│   03 Workflows — scripts that call primitives across     │
│                   instances. Needs the floor below.      │
├─────────────────────────────────────────────────────────┤
│ FLOOR 2 (the two tools)                                  │
│   02 Token Runtime  │  04 Registry                       │
│   (atomic moves)    │  (phone book)                      │
├─────────────────────────────────────────────────────────┤
│ FLOOR 1 (the foundation — build these FIRST, in parallel)│
│   01 ULIP            │   07 Data Model                   │
│   (the language)     │   (the database tables)           │
└─────────────────────────────────────────────────────────┘

           ▲ Elevator running through every floor:
           06 Non-Compliance + Observability
           (cross-cutting — touches every layer)
```

### Read it as a story

**Floor 1 — Foundation (build first, can be done in parallel):**
- **01 ULIP** is the *language* two instances speak. Without it, they can't say anything to each other.
- **07 Data Model** is the *database tables*. You need somewhere to store new state before you can write code that produces it.

> Nothing else can start until both of these are stable.

**Floor 2 — The two tools (build next, also in parallel):**
- **02 Token Runtime** = the atomic moves (`lock`, `commit_debit`, `create_incoming`, ...). It needs **01** (to verify signed envelopes) and **07** (new lock-related tables).
- **04 Registry** = the federation's phone book. It needs **01** because `RegisterName` / `ResolveName` / `Capabilities` are ULIP methods.

**Floor 3 — The orchestrator:**
- **03 Workflows** = "recipes" that call primitives across instances. To exist, it needs all of: **01** (envelopes), **02** (primitives to call), **04** (to look up the peer), **07** (workflow tables). That's why it's a floor above — it consumes everything below.

**Floor 4 — A feature on top of workflows:**
- **05 Migration** is a workflow that uses **03** and writes a forwarding pointer to **04**.

**Floor 5 — The roof:**
- **08 Tests** — a full two-instance test harness only makes sense once you have something to test end-to-end. Drafted early so test contracts shape the build, finalised last.

**The elevator (cross-cutting):**
- **06 Non-Compliance + Observability** isn't a single floor — it threads through everything. It archives **01** envelopes, lives in **04**'s lifecycle state, watches **03** workflows, and feeds **08** tests. Build it alongside the other floors, not as a separate slice.

### One-sentence version

> Build the **language** (01) and **tables** (07) first → then the **runtime** (02) and **registry** (04) → then **workflows** (03) that use them → then **migration** (05) on top → with **observability** (06) woven through and **tests** (08) as the cap stone.

---

## 5. The 9 slices — **"building a car in your garage, piece by piece"**

### The big idea (read this first)

The team is **not** building the entire federation at once. They're building it in **9 small steps**. Each step adds **ONE thing** and proves it works before the next step starts.

Imagine building a car in your garage. You don't start with the leather seats and the radio. You start with **wheels that turn**, then add brakes, then doors, then the engine, then paint. Each step, the car can do "one more thing."

### The logic of the slice order (the WHY)

Read the slices grouped like this and the order makes sense:

| Goal | Slices |
|---|---|
| **Make it work** | 0 (happy path) → 1 (handles failure) |
| **Make it secure** | 2 (instances trust each other) → 3 (users hold their own keys) |
| **Make it discoverable** | 4 (real phone book replaces hardcoded list) |
| **Add the big features** | 5 (account portability) → 6 (bridge to blockchains) |
| **Make it governable** | 7 (catch bad actors) |
| **Make it production-ready** | 8 (real deployment) |

⚠️ marks a **trust-boundary change** — get it wrong and the whole network breaks. Each ⚠️ needs a security review before moving on.

---

### Slice 0 — "Just prove tokens can MOVE between two boxes." ✅ Done

**Pretend you're starting from scratch.** You have two laptops, each pretending to be a UNITS instance. **Question**: can they actually pass tokens to each other? You don't know yet.

So you do the **bare minimum**:
- Two pre-configured users (alice on laptop A, bob on laptop B) — no signup yet
- A simple "send 100 tokens" command
- Both laptops blindly trust each other — no signatures checked
- The laptops find each other via a hardcoded YAML file

**Why this slice exists**: you're proving the *concept*. Not security. Not failure handling. Just: "do the wires connect? Do the tokens flow?"

**Done when**: alice sends 100 tokens to bob; both ledgers add up.

---

### Slice 1 — "What if laptop B unplugs mid-transfer?" ✅ Done

The happy case works. But what if **the network blinks** halfway through a transfer? Right now: alice's tokens are gone (debited), but bob never got them. **Money disappears.** Unacceptable.

So you add:
- A **retry loop** — "try again in 1s, then 2s, then 4s..."
- A way to **automatically reverse** the transfer if it really can't go through
- A "STUCK" state that flags broken transfers instead of silently failing them

**Done when**: pull the plug on laptop B mid-transfer → alice's tokens automatically come back.

---

### Slice 2 ⚠️ — "How do laptops actually TRUST each other?"

So far the laptops trust each other blindly. That's fine for testing — disastrous in production. **Anyone** could pretend to be a UNITS instance and send fake "send tokens" commands.

So you add:
- A cryptographic ID for each instance (like an instance "passport")
- Every message between instances is **signed**, and the receiver checks the signature
- An **allowlist** — which instances are even allowed to talk to you?

**Why ⚠️**: this is the first trust-boundary change. Get it wrong, the whole network is hackable. Security review before moving on.

**Status today**: half done. App-layer signing works (Ed25519). mTLS (network-level encryption) still pending. STRIDE review still pending.

---

### Slice 3 ⚠️ — "Now make REAL users do this, with their OWN keys."

Until now, the **instance** signs on behalf of the user — like a bank teller signing for you. Real federation needs **the user to own their key**, like owning your own house keys.

So you add:
- A mobile wallet (Silence Labs MPC) and a desktop wallet (Wagmi)
- A proper signup flow (interesting chicken-and-egg: you need a key BEFORE you can sign up — that's why signup moved to this slice)
- The user signs transfers themselves; the instance just relays the signed message

**Why ⚠️**: now users hold their own keys. Broken wallet = lost money. Another security review.

---

### Slice 4 — "Replace the YAML file with a real PHONE BOOK." ✅ Done

Remember the hardcoded YAML from Slice 0? Fine for 2 laptops. **Useless** for dozens of instances run by different companies.

So you replace the YAML with a real phone book — built on **DNS** (the same system that maps `google.com` to an IP address):
- Each instance registers itself by name
- Other instances look up names to find each other
- Each instance also publishes a "capabilities" doc: "I support these token types, these workflows, these protocol versions"

**Done when**: two laptops on the same Wi-Fi discover each other through a shared DNS server. **Zero config files.**

---

### Slice 5 — "Let alice MOVE her account from one instance to another." ✅ Done

What if alice doesn't like instance A anymore? She should be able to **switch providers** without losing her name, ID, or balance. Like keeping your phone number when you change carriers.

So you add:
- A migration workflow that ships alice's state from A to B
- A **forwarding pointer** on A (like mail forwarding) for a grace period
- A user-signed consent claim — instances can't just move accounts without permission

**Done when**: alice's account moves A→B; her DID is unchanged; balance intact on B; anyone still sending to A gets forwarded.

---

### Slice 6 — "Let alice pay USDC on Ethereum and receive NATIVE tokens." ✅ Done

UNITS isn't an island. It needs to **bridge to real blockchains** (Ethereum, Solana) so users can move money in and out.

So you add:
- A "swap" workflow: takes USDC on Ethereum, gives the user native UNITS tokens
- The **reverse**: native tokens → USDC out
- A new "**Token Manager**" service (doesn't exist today) that tracks the on-chain side
- An operator treasury (multi-sig) that backs the swaps

**Done when**: alice swaps 10 USDC → 100 NFH-T; reverse works; failures are cleanly handled (pre-confirmation: abort; post-confirmation: operator compensates).

---

### Slice 7 — "Add CCTV cameras and a HEALTH INSPECTOR." ✅ Done

The federation works. But what if one instance **misbehaves** — lies about balances, censors transactions, gets hacked? Right now: nobody would notice.

So you add:
- Every signed message is **archived** (full auditable record)
- Each instance pushes telemetry to a central observability layer
- A **lifecycle for misbehaving instances**: Watch → Restricted → Quarantined → De-listed
- A "**cross-witness**" job: if two instances disagree about what happened, alert
- **Forced reconstruction**: if an instance is de-listed, governance can rebuild its affected accounts on a healthy peer using the archived signatures

**Done when**: misbehaving instance gets quarantined; affected accounts get reconstructed safely.

---

### Slice 8 ⚠️ — "Open the doors for REAL customers." Not started

Everything works in the lab. Now it needs to work in **production** — under real load, real money, real datacenters.

So you add:
- Helm charts (Kubernetes deployment recipes) for every service
- Terraform for Cloud DNS, Cloud KMS, Vault Transit
- Multi-environment configs (dev / staging / prod)
- **Rehearsed runbooks** for every "what if?" scenario
- Performance + chaos test suites green in CI

**Why ⚠️**: final security + ops review before going live. No "undo" button after this.

---

### Recap in 9 sentences

1. **Slice 0**: prove tokens move at all (stubbed everything).
2. **Slice 1**: recover when the network blinks.
3. **Slice 2 ⚠️**: instances cryptographically prove who they are.
4. **Slice 3 ⚠️**: users hold their own keys, not the instance.
5. **Slice 4**: real DNS-backed phone book replaces the YAML.
6. **Slice 5**: accounts can move between instances.
7. **Slice 6**: bridge to Ethereum / Solana for real money in/out.
8. **Slice 7**: observability + handle bad actors.
9. **Slice 8 ⚠️**: ship to production for real customers.

---

## 6. Cross-cutting workstreams — **"continuous chores, not slices"**

Some work doesn't belong to any one slice — it runs **alongside** every slice, like a janitor walking the halls. If you tried to schedule it as its own slice, the system would be broken in between.

**Mental model**: think of the slices as floors of a building and these as the plumbing/wiring/fire-alarms — threaded through every floor as it goes up, never built as their own floor.

| Workstream | Touched when | Why it's cross-cutting |
|---|---|---|
| OpenAPI specs (06a) | any endpoint changes | API contracts must stay in sync with code — can't batch |
| Per-service READMEs | every slice | docs go stale instantly otherwise |
| DB migration phasing (07a) | every slice that adds columns | schema is the contract — needs phased rollout, not big-bang |
| E2E test fixtures (12a) | every "Done when" criterion | tests for slice N must exist before slice N is declared done |
| Threat-model review (11a) | slices 2, 3, 7, 8 | any auth or governance change forces a re-look |

---

## 7. Build order **inside** one slice — **like cooking a meal**

When you're working on a slice, do the steps in this order. The classic mistake is building the UI first and discovering the DB columns don't exist.

| Step | What | Cooking analogy |
|---|---|---|
| 1 | Re-read the spec; capture decisions | mise en place — read the recipe |
| 2 | **DB migrations** — schema is the contract | prep the workspace |
| 3 | Token Runtime — new primitives | the base / sauce |
| 4 | Registry changes (when applicable) | the lookup ingredients |
| 5 | Workflows — orchestrate the primitives | combine and cook |
| 6 | units-api — expose endpoints | plating |
| 7 | finternet-app UI (**only Slice 3+**) | garnish |
| 8 | E2E test for the "Done when" criterion | taste-test |
| 9 | Docs + runbook | write down the recipe |

Steps 2–6 can run in parallel where different services are owned by different people.

---

## 8. Gates between slices — **"exams before the next grade"**

Before you're allowed to start slice N+1, **all** of these must be true for slice N:

- ✅ "Done when" criterion **passes in CI**
- ✅ Migrations **deployed in staging**
- ✅ Open issues **triaged**
- ✅ **Rollback path** documented and tested
- ✅ Security review (only for trust-boundary slices: **2, 3, 7, 8**)

**Why this is strict**: federation has no "rollback to last week" button — every slice changes the wire protocol or the schema. If slice N is half-done, slice N+1 will be built on quicksand.

---

## 9. Team ownership — **who builds what in parallel**

Eight specs split cleanly onto **four teams** so they can work in parallel after the foundation (Floor 1 from §4) is laid.

| Team | Owns | Why these specs together |
|---|---|---|
| **Protocols** | 01 (ULIP), 04 (Registry) | both are about wire format / discovery — same skill set |
| **Runtime** | 02 (Token Runtime), 07 (Data Model) | Rust code + DB schema live together |
| **Workflow** | 03 (Workflows), 05 (Migration) | migration *is* a workflow — same Restate/TS skills |
| **Platform / SRE** | 06 (Non-compliance + Obs), 08 (Test + Rollout) | observability + deployment + chaos = ops |

Each spec ships with a file-path-level change list, new types + signatures, test acceptance criteria, and explicit "seams to other specs" so cross-team coordination doesn't go off-script.

---

## 10. Out of scope — **things we are explicitly NOT doing**

Three temptations the architecture doc deliberately defers. If a stakeholder asks for any of these, the answer is **"after Slice 8 ships, we re-scope."**

| Excluded                                                                      | Why it's out                                                                                |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `proxy_swap` cross-class / cross-chain variants (DEX-style, bridge, compound) | one extra dimension of risk per variant; ship simple swap first                             |
| Multi-hop routing (A → B → C)                                                 | introduces routing tables and trust-chains; not needed until there are >2 instances in prod |
| Consensus-backed registry alternatives                                        | DNS is good enough and ships now; blockchain-as-registry is a multi-year detour             |

---

## 11. Where each piece lives in the current codebase — **a map**

"The codebase as it is **today**, and what this work **changes** about each address."

### `units-token-runtime/` (Rust)
- **Today**: lock state has `locked, locked_by, locked_until` — **no `lock_id`**. Ops are chunky: `Mint, Burn, Transfer, Freeze, Lock, Update, AddCredential, Generic`. Kafka msg has `tx_id, signature` but signature **never verified**.
- **After**: Spec 02 adds `lock_id` and atomic primitives. Spec 01 turns on signature verification. Spec 07 adds supporting tables.

### `units-workflows/` (TS / Restate)
- **Today**: HTTP envelope is `{context, payload}` — **no signature field**. Workflows exist for `gateway, profile-update, delegation` (signal-driven via `ctx.promise(name).orTimeout(ms)`).
- **After**: Spec 01 promotes the envelope to a **signed ULIP envelope**. Spec 03 adds `signup, transfer, swap, migration` apps.

### `units-api/` (Go / Fiber)
- **Today**: Account model has `address, DID, entity_type, vault_entity_id, PII` — **no `instance_namespace`**. Restate, Kafka, Keycloak OIDC, OPA already wired.
- **After**: Spec 07 adds `instance_namespace`. Spec 04 adds `RegisterName`/`ResolveName` endpoints. Spec 06 adds lifecycle-state columns.

### `units-services/`
- **adapterOrchestrator** (Go) — stateless proxy routing by `context.chainId`. Spec 04 makes it instance-aware.
- **notificationService** (Node) — OTP + JWT. **Unchanged** by federation.
- **proofService** (Rust) — **spec-only today, not implemented**. Spec 06 fills it in as the audit-anchor service.
- **registry** — **doesn't exist yet**. Slice 4 builds it in `units-services/registry/` (not its own repo).

### `units-adapters/chain/alchemyAdapter/` (Go)
- **Today**: already publishes full lifecycle on Kafka (`finternet.transactions.{submit,confirm,status,dlq}`). EVM + Solana. Adapter never holds keys — client-side signing.
- **After**: **adapter itself is unchanged**. Spec 02 + Slice 6 add a brand-new **Token Manager** service that consumes `transactions.status` and writes proxy ledger entries — a service that doesn't exist today.

### `finternet-app/` (Next.js 15)
- **Today**: OTP login flow, `NEXT_PUBLIC_UNITS_API_BASE` hardcoded to one instance, no `name@instance` addressing, no cross-instance flows. Wagmi (EVM) + Solana Web3.js + Silence Labs MPC wallets exist but **no UNITS-side signing**.
- **After**: Spec 04 adds an instance picker. Spec 03 makes transfer UI use `name@instance`. Spec 05 adds migration UI. Spec 09 wires user-held keys into signing flows.

### `units-ulip/` (new sibling repo — **doesn't exist yet**)
- Created by Spec 01. Holds the authoritative `.proto`s, canonical-serialisation rules, conformance test suite, and reference SDKs (Go / Rust / TS) published as packages. **Everything else depends on the SDK packages**, never on local protocol source.

### Schemas
- **Today**: JSON Schemas at `specs/schemas/{account,token,token-class,transaction,credential,core}/`. OpenAPI at `units-api/specs/api/*.yaml`. DB DDL at `units-api/specs/db/` + `units-token-runtime/specs/db/`. **No `.proto`, no signed-envelope schema, no DNS record schema, no capability document schema.**
- **After**: Spec 01 puts `.proto` in the new ULIP repo. Spec 04 adds DNS-record + capability schemas in `specs/schemas/`.

### Infra
- **Today**: Helm charts for units-api, finternet-app, keycloak, vault, yugabyte, kafka, minio, clickstack, ingress-controller, cert-manager. ArgoCD wave-ordered deployments. Terraform for GKE + Cloud KMS. MCP server (`units-mcp/`).
- **Federation-shaped gaps today**: no DNS registrar, no capability directory service, no inter-instance mTLS mesh, no aggregated cross-peer observability. Slices 4 / 7 / 8 build these.
---
## 13. The 60-second elevator pitch (when someone asks "what is this?")

> UNITS today is one bank. Federation makes it many banks that talk to each other over a signed envelope protocol called ULIP, find each other through a DNS-backed registry, and prove every transfer end-to-end with chained SHA-256 commitments. The Rust token runtime gets split into atomic primitives (`lock`, `commit_debit`, `create_incoming`, `commit_credit`, plus reversals) so a transfer can pause halfway and either complete on the other side or cleanly reverse. Workflows on Restate orchestrate it. Accounts can migrate between instances with a forwarding pointer. Misbehaving instances move through Watch → Restricted → Quarantined → De-listed. We're shipping in vertical slices: skeleton → failure paths → real auth → user keys → DNS → migration → swap → observability → production. Slice 2 application layer is live; mTLS and security review are the current gates.

