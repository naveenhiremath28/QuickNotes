## Proof Service

```md
Imagine the Finternet platform is a bank. Every time money (tokens) moves, the bank writes it down in its ledger (the database). But here's the question: how do you prove later that nobody secretly edited the ledger?

That's the proofService's whole job — it's like a notary that stamps batches of transactions so they can't be silently changed afterwards.

What Happens When You Make a Transaction

Step 1 — You transfer a token
You hit "transfer" in the app. The request flows through the system (app → units-api → Kafka → tokenEngine), and the token engine writes the result into the database:

▎ "Transaction #123: Alice sent 50 tokens to Bob — done."

Importantly, the row gets saved with a little flag: proof_status = 'pending'. That flag means: "this transaction hasn't been notarized yet."

At this point, the proofService hasn't done anything. Your transfer is already complete — you don't wait for it.

Step 2 — The proofService wakes up (every ~60 seconds)
It runs in the background on a timer, like a night-shift clerk. It asks the database:

▎ "Give me all transactions still marked 'pending'."

Say it finds 1,000 of them.

Step 3 — It bundles them into a "fingerprint tree" (Merkle tree)
Think of it like this:
- Each transaction gets a unique fingerprint (a hash — change even one character of the transaction, and the fingerprint completely changes)
- Fingerprints are paired up and combined, again and again, like a tournament bracket
- At the top you get one single fingerprint that represents all 1,000 transactions — the "Merkle root"

        ROOT  ← one fingerprint for the whole batch
       /    \
   hash      hash
   /  \      /  \
 tx1  tx2  tx3  tx4 ...

If anyone later edits any of those 1,000 transactions, even by one digit, the ROOT won't match anymore. Tampering becomes instantly detectable.

Step 4 — It writes the receipts back to the database (the DB writes you asked about)                                                                                                                Two things happen:
1. INSERT into proofs table — for each transaction, it saves a "receipt": "here's your fingerprint, here's your position in the tree, here's the path to the root"                                  2. UPDATE the transactions table — flips the flag from pending → proven, so the same t twice
                                                                                                                                                                                                    Step 5 — Later, anyone can verify
Months later, an auditor (or Bob, who's suspicious) can call the API:                                                                                                                               
▎ "Show me the proof for transaction #123."                                                                                                                                                         
The service returns the receipt, and anyone can independently do the math: take the transaction, hash it, follow the path up the tree — if it lands on the same ROOT, the transaction is guaranteed untampered. If it doesn't match, someone messed with the data.

In One Sentence

▎ Your transfer happens normally and instantly; proofService quietly comes along aftertions into a tamper-proof fingerprint, saves a verifiable receipt for each one, andmarks them as "proven" in the database.

```



```
================================================================
   FINTERNET — proofService EXPLAINED
   ("How does the bank prove nobody edited the ledger?")
================================================================

----------------------------------------------------------------
1. THE BIG ANALOGY
----------------------------------------------------------------

Imagine the Finternet platform is a BANK.
  - Every time tokens move → the bank writes it in its ledger
    (the database).
  - The ledger is the source of truth.

THE QUESTION:
  How do you prove, later, that nobody secretly edited the
  ledger?

THE ANSWER:
  proofService = a NOTARY that stamps batches of transactions
                 so they CANNOT be silently changed afterwards.

  Once stamped, any future tampering becomes mathematically
  detectable.

----------------------------------------------------------------
2. THE CAST (PLAYERS)
----------------------------------------------------------------

  ┌──────────────────────────────────────────────────────────┐
  │  finternet-app (Next.js + Express)                       │
  │     The app Alice uses                                   │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  units-api (Go)                                          │
  │     The front desk — validates, signs, records           │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  Vault (via key_references)                              │
  │     The safe where private keys live                     │
  │     Keys never leave it                                  │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  Kafka                                                   │
  │     The conveyor belt of operations                      │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  tokenEngine (Rust)                                      │
  │     The vault clerk — actually moves the tokens          │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  proofService (Rust)                                     │
  │     The notary — comes in ONLY at the end                │
  └──────────────────────────────────────────────────────────┘

----------------------------------------------------------------
3. THE TABLES
----------------------------------------------------------------

  ┌──────────────────────────────────────────────────────────┐
  │  accounts                                                │
  │     Owner       : units-api                              │
  │     Holds       : Alice & Bob's accounts + their DID     │
  │                   (digital identity)                     │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  key_references                                          │
  │     Owner       : units-api                              │
  │     Holds       : Pointers to keys in Vault + public     │
  │                   keys (private keys are NOT in the DB)  │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  transactions                                            │
  │     Owner       : units-api                              │
  │     Holds       : The envelope: "transfer requested",    │
  │                   status, proof_status                   │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  token_transactions                                      │
  │     Owner       : tokenEngine writes                     │
  │     Holds       : The actual ledger entries —            │
  │                   debit/credit, signature (JSONB),       │
  │                   state hashes                           │
  └──────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────┐
  │  proofs                                                  │
  │     Owner       : proofService writes                    │
  │     Holds       : The notarized Merkle receipts          │
  └──────────────────────────────────────────────────────────┘

----------------------------------------------------------------
4. WHAT HAPPENS WHEN YOU MAKE A TRANSACTION
----------------------------------------------------------------

STEP 1 — You transfer a token
  You hit "transfer" in the app. The request flows:
      app → units-api → Kafka → tokenEngine

  The tokenEngine writes the result into the database:
      "Transaction #123: Alice sent 50 tokens to Bob — done."

  Importantly, the row is saved with a flag:
      proof_status = 'pending'

  Meaning:
      "This transaction has NOT been notarized yet."

  IMPORTANT:
  - The proofService hasn't done anything yet.
  - Your transfer is ALREADY COMPLETE.
  - You don't wait for the proof step.

----------------------------------------------------------------
5. THE FULL "ALICE SENDS 100 TO BOB" FLOW
----------------------------------------------------------------

STEP 1 — Alice taps "Send 100 to Bob"
  finternet-app
    → Next.js
    → Express BFF (:6000)
    → proxied to units-api (:3000)

STEP 2 — Identity & signing (units-api + Vault)
  - Alice was onboarded earlier:
      - accounts row → has her DID (did:web:alice)
      - key_references → "Alice's key #1 lives in Vault
                          under this name; here is her
                          public key."
  - units-api's KeyManager asks Vault:
      "Sign the hash of this operation with Alice's key."
        → Sign(ctx, ref, hash)
  - Vault signs INSIDE the safe and returns only the signature.
  - The private key NEVER touches units-api or the database.
      → A DB attacker cannot forge it.

STEP 3 — Record + hand off (units-api → transactions → Kafka)
  - units-api inserts a row into transactions:
      status        = pending
      proof_status  = 'pending'    ← future to-do list
  - It publishes a TokenOperationMessage to Kafka
      topic: units.token.operations
      payload: operation + Alice's identity + signature
  - Alice instantly gets back a transaction ID.
  - She does NOT wait.

STEP 4 — Tokens actually move
  tokenEngine consumes the Kafka message and writes the ledger:
  - Two rows in token_transactions (double-entry):
      debit  Alice  −100
      credit Bob    +100
  - Each row stores:
      state_before  / state_after   (hash chain links)
      signature column              (Alice's signature)
  - Marks the transactions envelope completed.
  - Emits an audit event to Kafka.

  ⏱  At this moment, the transfer is DONE.
      proofService still hasn't appeared.

----------------------------------------------------------------
6. THE proofService — WHAT IT DOES (in plain words)
----------------------------------------------------------------

STEP A — Wakes up every ~60 seconds
  Runs in the background like a NIGHT-SHIFT CLERK.
  It asks the database:
      "Give me all transactions still marked 'pending'."

  Say it finds 1,000 of them.

STEP B — Bundles them into a FINGERPRINT TREE (Merkle tree)
  Think of it as a tournament bracket of hashes.

  - Each transaction gets a unique FINGERPRINT (a hash).
      Change even one character → the fingerprint changes
      completely.
  - Fingerprints are paired and combined, repeatedly.
  - At the top: ONE single fingerprint = the "Merkle root".

  Visual:

            ROOT          ← one fingerprint for the whole batch
           /    \
         hash    hash
         /  \    /  \
       tx1 tx2 tx3 tx4 ...

  KEY PROPERTY:
  If anyone later edits ANY of those 1,000 transactions,
  even by one digit, the ROOT won't match anymore.
  → Tampering becomes INSTANTLY detectable.

STEP C — Writes receipts back to the database
  (These are the actual DB writes that proofService makes.)

  Two things happen:
    1. INSERT INTO proofs
         For each transaction, save a "receipt":
           - the fingerprint (leaf hash)
           - the position in the tree
           - the path to the root
    2. UPDATE transactions
         Flip the flag: pending → proven
         So we don't notarize the same transaction twice.

STEP D — Later, anyone can verify
  Months later, an auditor (or suspicious Bob) can call:
    "Show me the proof for transaction #123."

  The service returns the receipt. Anyone can independently
  do the math:
    take the transaction
    → hash it
    → follow the path up the tree
    → check it lands on the same ROOT

  If it matches → transaction is GUARANTEED untampered.
  If it doesn't → someone messed with the data.

----------------------------------------------------------------
7. WHERE proofService ENTERS — 3 PLACES
----------------------------------------------------------------

ENTRY #1 — The background notary (~every 60s)
  proof_generator.rs polls:
      SELECT ... FROM transactions
      WHERE proof_status = 'pending'
  Finds Alice's transaction + up to 999 others.
  Reads their token_transactions rows (read-only).
  Builds the BLAKE3 Merkle tree.

ENTRY #2 — Writing the receipts (its only DB writes)
  INSERT INTO proofs
      Alice's transaction gets its Merkle receipt
      (root, path, leaf hash).
  UPDATE transactions
      SET proof_id = ..., proof_status = 'proven'
      → crossed off the to-do list.

ENTRY #3 — Anyone asks for proof later
  POST /v1/transaction/proof
      "Give me Alice's receipt."
  POST /v1/transaction/proof/verify
      "Check this receipt is mathematically valid."

----------------------------------------------------------------
8. FULL SYSTEM FLOW — VISUAL
----------------------------------------------------------------

  Alice
    │
    ▼
  finternet-app
    │
    ▼
  units-api ──────┬──► Vault (signs with Alice's key)
                  │
                  ├──► transactions table
                  │      proof_status = 'pending'
                  │
                  └──► Kafka
                         │
                         ▼
                       tokenEngine
                         │
                         ▼
                       token_transactions
                         (debit/credit + hash chain)

  ⏱  ~60 seconds later...

  proofService
    │
    ├──► Reads pending transactions
    ├──► Builds Merkle tree
    ├──► INSERT INTO proofs
    └──► UPDATE transactions
           SET proof_status = 'proven'

----------------------------------------------------------------
9. KEY INTUITION SUMMARY
----------------------------------------------------------------

  THE TRANSFER and THE PROOF are SEPARATE:
    - Transfer happens INSTANTLY (Alice → Bob).
    - Proof happens LATER, in the background (~60s).
    - Alice never waits for the proof step.

  WHY THE PROOF MATTERS:
    - The DB alone could be edited by an insider/attacker.
    - The Merkle root is a tamper-evident "fingerprint" of
      a whole batch of transactions.
    - One byte changed → the whole root mismatches.

  WHAT proofService ACTUALLY DOES:
    1. Polls pending transactions every ~60s
    2. Builds a Merkle tree (BLAKE3 hashes)
    3. INSERTs receipts into proofs
    4. UPDATEs proof_status from pending → proven
    5. Serves /proof and /proof/verify on demand

  PRIVATE KEYS:
    - Live in VAULT
    - Never leave the safe
    - DB attackers cannot forge Alice's signature

  ONE-LINER:
    "Your transfer happens instantly. proofService quietly
     comes along after, bundles transactions into a
     tamper-proof fingerprint, saves a verifiable receipt
     for each one, and marks them as 'proven' in the
     database."

================================================================
```

---
---

# **Units Federation**

Notes in:
/Users/naveenvhiremath/Downloads/finternet/federation/

```bash
Resume this session with:
claude --resume f925f63f-e0ef-4d4a-aca8-3b387b8424be
```
## Alice sends Bob 100 tokens

1. alice@a.local → bob@b.local, 100 NFH-T tokens
2. Alice asks her bank to send money -> POST /v1/token/transfer
3. Bank A looks Bob up in the phone book -> Bank A asks the **Registry**: "who is bob?"
4. Bank A writes the to-do list and hands it to the checklist runner -> PrimitivePlan → Restate workflow (**4-step plan**)
#### what is ULIP?
**ULIP is the standardized order form the banks use** — like SWIFT or UPI message formats in real banking. Every step order below (Lock, CreateIncoming, CommitDebit, CommitCredit) is a **ULIP envelope**: a fixed-format message, **signed with the sending bank's key**, posted to the receiving bank's front-door mail slot (`/v1/ulip/<Method>`).

5.  Alice immediately gets back: **"202 — accepted, here's your tracking number."**
6. Step 1 — LOCK: freeze 100 in Alice's account -> `/v1/ulip/Lock → Kafka → Engine A`
```
Alice: 900 available + 🔒 100 locked
Bob: 0
```
7. Step 2 — CREATE INCOMING: Bank B reserves a slot for Bob
```
Alice: 900 + 🔒 100
Bob: 0 + 📥 100 reserved
```
**Receipt goes BACK to Bank A**
 
 **AFTER THE NEXT STEP THERE'S NO "CANCEL", ONLY "FINISH" OR "REVERSE"**
 
8. Step 3 — COMMIT DEBIT: the 100 actually leaves Alice
```
Alice: 900
Bob: 0 + 📥 100 reserved
```
9. Step 4 — COMMIT CREDIT: Bob gets his money
**Engine B converts the reserved 100 into Bob's real, spendable balance.**
```
Alice: 900 ✓
Bob: 100 ✓
```
### ✅ Done — COMMITTED
Alice −100 at Bank A · Bob +100 at Bank B · Four steps, four signed receipts, one boss.  
Alice can check status anytime with her tracking number.

## When things go wrong

1. BEFORE step 3 → ABORTED
	## Fails early? Just unfreeze.
		The workflow runs Unlock, Alice's 100 is unfrozen, 
		status = ABORTED
2. AFTER step 3 → REVERSE
	### Fails late? Automatic refund.
		Bank A **asks Bank B directly: "did the credit land?"** If the answer is
		a definite _no_, the workflow runs the undo steps: **re-credit Alice at
		A, cancel the reservation at B**. Alice ends net-zero.
		Status = **REVERSED**
3. Can't prove it either way? Stop and page a human.
	If Bank B's answer is **"unknown"**
	 **refuses to guess** — status = **STUCK**, an operator investigates



```
  ---
  Signup (OTP-first)

  The key change: you no longer need a crypto key to sign up. Trust comes from proving you own an email/phone via OTP. The instance provisions a custodial key for you.

  Step 1 — Send OTP
    User → Registry: POST /v1/account/login { username: "alice@example.com" }
    Registry → OTP service: generate OTP   (no otp in payload = "send" mode)
    ← { success: true, "OTP sent" }

  Step 2 — Verify OTP + existence check
    User → Registry: POST /v1/account/login { username, otp: "123456" }
    Registry → OTP service: verify → issues a JWT (iss="registry", RS256)
    Registry → DB: does SHA-256(email) exist in registered_names?

      If NEW user → returns instance picker:
        { is_existing: false, instances: [ a.local, b.local, ... ] }
        User picks an instance → Step 3

  Step 3 — Create account (at the chosen instance, synchronous)
    User → Instance: POST /v1/account/create
      { payload: { address:"alice", homeInstance:"a.local", ... } }
      Header: Bearer <registry JWT>

    Instance does, in order:
      1. Validate the OTP JWT (RS256, iss=registry)
      2. Call registry to RESERVE the name server-side
      3. Registry returns a SIGNED reservation proof (90s TTL)
      4. Verify proof signature against pinned REGISTRAR_PUBKEY (fail-closed)
      5. Check the reservation hasn't expired
      6. Create a Vault Transit Ed25519 key
      7. Derive DID = did:units:0x<hex(vault_pubkey)>
      8. INSERT accounts (status='active', did, home_instance)
      9. INSERT key_references (key_source='vault_transit')
     10. Encrypt PII (email) under a Vault entity key
     11. Create the Keycloak auth user
    ← { accountId, did, status: "active" }

  Why it matters: there's no Restate workflow and no status polling anymore — it's all synchronous, because the registry has already verified the user up front via OTP. The 90s signed reservation closes
  the name-squatting race that existed before.

  ---
  Login (two paths)

  Login depends on where the user starts.

  Path A — Start at the registry (new, the ADR-007 path)

  1. User → Registry: POST /v1/account/login { username }              → OTP sent
  2. User → Registry: POST /v1/account/login { username, otp }
       Registry verifies OTP → JWT
       Registry looks up SHA-256(email) in registered_names
       ← { is_existing: true, home_instance: "a.local", access_token: <JWT> }

  3. User → Instance: POST /v1/account/login { username, ssoLogin: true }
       Header: Bearer <registry JWT>
       Instance → SSOJWTVerifier: fetch registry pubkey from /.well-known/ulip.json,
                  verify the JWT signature, extract username from VERIFIED claims
                  (never trusts the request body)
       Instance → findAccountByUsername → Keycloak LoginByTokenExchange
       ← { access_token: <Keycloak JWT> }

  The registry acts as the SSO identity provider; the instance trades that verified identity for a local Keycloak session.

  Path B — Start directly at the instance (existing flow, still works)

  4. User → Instance: POST /v1/account/login { username }
       Instance → OTP service (registry's global one)                  → OTP sent
  5. User → Instance: POST /v1/account/login { username, otp }
       Instance verifies OTP → findAccountByUsername
          • found locally    → Keycloak token exchange → local JWT
          • NOT found locally → resolveViaRegistry → FORWARD response:
              { code:"FORWARD", home_instance:"a.local",
                api_url:"http://units-api-1:3000" }
            Client retries login at the correct instance.

  So if Alice's account lives on a.local but she hits b.local, the instance doesn't error — it tells her where to go.

  ---
  The two new trust mechanisms underneath both flows

  6. /.well-known/ulip.json — every instance and the registry publish their signing keys here. Verifiers fetch it (cached 1h) to check signatures by key_id. This is how the instance verifies the
  registry's JWT in Path A instead of blindly trusting it.
  7. Pinned registrar pubkey — units-api now verifies the registrar's signature on the reservation proof (signup) and on all resolve paths (ResolveByAddress + ResolveByDID), fail-closed.

  ---
  A caveat worth flagging since you're verifying this: the docs say the OTP JWT issuer is "registry" using RS256, but several other pieces (notification service, ULIP envelopes) moved to Ed25519/EdDSA.
  The JWT signing algorithm is mid-migration, so the verifier in Path A has to handle both — that's a likely spot for bugs.

```