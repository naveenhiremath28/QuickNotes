# Diagrams — Finternet Platform

All diagrams in one place (mermaid). Per-service context lives in the numbered docs.

## 1. Platform Block Diagram

```mermaid
flowchart TB
    subgraph Client
        B[Browser]
        AI[AI Agents]
    end

    subgraph fapp ["finternet-app"]
        APP["Next.js :3002"]
        BFF["Express BFF :6000<br/>oidc · sso · wallet · kyc · allowlist proxy"]
    end

    MCP["units-mcp<br/>(10 read-only tools)"]

    subgraph instance ["Home instance (one of N)"]
        API["units-api :3000"]
        PG[(PostgreSQL)]
        K[[Kafka]]
        TE["tokenEngine"]
        TP["tokenPrograms"]
        PS["proofService"]
        VA["Vault"]
        KC["Keycloak"]
    end

    subgraph shared ["Federation-shared"]
        CR["central registry :3000/:3001"]
        OTP["notificationService :4000"]
        RS["Restate :8082"]
        WF["units-workflows :9080"]
    end

    subgraph chain ["Chain access"]
        AO["adapterOrchestrator :8090"]
        CA["alchemyAdapter :3500"]
        RPC["Blockchain RPCs (Alchemy)"]
    end

    WPBE["wpbe-silencelabs :8090"] --- SN["Silent Network nodes<br/>(MPC keyshares)"]
    PEER["peer instances"]

    B --> APP --> BFF
    BFF --> API & CR & WPBE
    AI --> MCP --> API
    API --> PG & VA & KC
    API -- outbox --> K --> TE --> TP
    TE --> PG
    TE -- signed completion --> API
    PS --> PG
    API <--> CR
    API --> RS --> WF
    WF -- ULIP + callbacks --> API
    WF -- ULIP --> PEER
    API -- "/v1/ulip/*" --> PEER
    API --> AO --> CA --> RPC
    CR --> OTP
    API -- OTP JWKS --> OTP
```

## 2. Cross-Instance Token Transfer (two-party prepare/commit saga)

```mermaid
sequenceDiagram
    participant U as Browser
    participant X as BFF
    participant S as units-api SOURCE
    participant R as Restate: primitive-operation
    participant D as units-api DEST
    participant K as Kafka (per instance)
    participant E as tokenEngine (per instance)

    U->>X: transfer
    X->>S: POST /api/v1/token/transact (signed envelope)
    S->>S: plan = program.plan() ?? two_party_prepare_commit@2.0<br/>write transactions.plan_data (planHash)
    S->>R: StartWorkflow(txnId)

    rect rgb(235, 245, 255)
    Note over R,E: op 1 — Lock (source)
    R->>S: POST /v1/ulip/Lock
    S->>S: admit → primitive_ops (dedup + command outbox)
    S->>K: TokenOperationEvent (key = txId)
    K->>E: execute fungible lock → locks[(txn,op)]
    E->>S: POST /v1/internal/primitives/complete (Ed25519-signed)
    S->>R: signal primitiveCompleted (signal outbox)
    end

    R->>D: op 2 — /v1/ulip/CreateIncoming (proofOfLock: op 1)
    Note over D: incoming[] entry on destination token
    R->>S: op 3 — /v1/ulip/CommitDebit (POINT OF NO RETURN, prunes op 1)
    R->>D: op 4 — /v1/ulip/CommitCredit (prunes op 2)
    Note over R: statuses: SUBMITTED → PREPARED(2) → COMMITTING(3) → COMMITTED(4)

    Note over R: FAILURE before op 3:<br/>RejectIncoming(dest) + Unlock(source) → ABORTED
    Note over R: FAILURE after op 3: destination-status query →<br/>COMMITTED → committed · NOT_COMMITTED → Credit(source)+RejectIncoming → REVERSED · UNKNOWN → STUCK
```

## 3. Engine Execution Pipeline (per Kafka message)

```mermaid
flowchart TB
    K[Kafka message] --> RT["retry wrapper (3 attempts)"]
    RT --> ID{federation op?}
    ID -- dedup hit --> CB[re-POST completion, done]
    ID --> P["status → processing"]
    P --> C["token_class_configs → program_id"]
    C --> G["standard + capability gate (fail-closed)"]
    G --> U["domain_lifecycle unwrap"]
    U --> A["load additional states<br/>(issuer / loans / credentials)"]
    A --> L["load or create TokenState"]
    L --> V1["verify commitment chain"]
    V1 --> V2["verify ULIP envelope<br/>(create_incoming / commit_credit)"]
    V2 --> H1["validate + pre-hooks<br/>(validation, min-balance, max-supply)"]
    H1 --> EX["program.execute → AffectedStates"]
    EX --> TX["ONE DB transaction:<br/>tokens (optimistic lock on state_version)<br/>+ token_transactions (debit/credit)<br/>+ state_history (commitment chain)<br/>+ primitive_ops CAS + signed completion"]
    TX --> H2["post-hooks (warn-only)"] --> AUD["audit_events"]
    TX -- error --> F["failed → DLQ if terminal"]
```

## 4. State Commitment Chain

```mermaid
flowchart LR
    subgraph v1 ["state_history v1"]
        C1["commitment₁ = sha256(<br/>prev='' ‖ tx₁ ‖ ts ‖ id ‖ owner ‖<br/>identities* ‖ relationships ‖ state ‖ v1 ‖ data)"]
    end
    subgraph v2 ["state_history v2"]
        C2["commitment₂ = sha256(prev=commitment₁ ‖ …)"]
    end
    subgraph v3 ["state_history v3"]
        C3["commitment₃ = sha256(prev=commitment₂ ‖ …)"]
    end
    C1 --> C2 --> C3
    T["tokens.state_commitment = commitment₃<br/>tokens.state_version = 3 (optimistic lock)"]
    C3 -.-> T
```

\* `identities` are filtered before hashing: delegation-managed types (`["access"]`) are excluded so grants/revokes don't invalidate prior commitments. Each `state_history` row stores the `commitment_config` used, so history stays verifiable after config changes.

## 5. Federation Routing at the BFF

```mermaid
flowchart TB
    RQ["Incoming /api/v1/* request"] --> AL{"in the 37-route allowlist?"}
    AL -- no --> R404[404]
    AL -- yes --> EC["enrichContext<br/>(fill ts/msgId/developerToken)"]
    EC --> MW{route chain}
    MW -- authenticated --> IU["instanceUrlTarget:<br/>decode JWT instance_url (unverified)<br/>→ exact-match registry catalog"]
    IU -- unknown/missing --> R401["401 FED_INSTANCE_UNRESOLVED"]
    IU -- validated --> FWD
    MW -- signup --> SI["signupInstanceTarget:<br/>payload.homeInstance UUID → catalog<br/>(signup-gated)"]
    MW -- login --> CT["contactTarget:<br/>payload.username → registry /v1/resolve"]
    SI --> FWD
    CT --> FWD
    FWD["forward: strip cookies,<br/>re-stream body, rewrite /api → ''"]
    FWD --> H1["units-api @ home instance A"]
    FWD --> H2["units-api @ home instance B"]
```

## 6. Auth & Login Flows

```mermaid
sequenceDiagram
    participant B as Browser
    participant N as Next.js (NextAuth + middleware)
    participant X as BFF
    participant G as Google
    participant CR as Central Registry
    participant A as units-api (home)

    Note over B,A: SSO login (ADR 0002 — BFF signs with the shared OTP key)
    B->>N: Sign in with Google
    N->>G: OAuth → id_token
    N->>X: POST /api/v1/auth/sso {idToken}
    X->>G: verify id_token (JWKS)
    X->>X: mint authToken (EdDSA, shared otp-service key)
    X->>CR: POST /v1/resolve (email → home)
    alt existing user
        X->>A: /v1/account/login (payload.authToken)
        A-->>B: Keycloak session JWT (instance_url claim)
    else new user
        X-->>N: authToken + isExisting:false → onboarding /account/create
    end

    Note over B,A: OTP login
    B->>X: POST /api/v1/account/login {username}
    X->>CR: contact resolve → home instance
    X->>A: forwarded login → OTP generate (notificationService)
    B->>X: verify OTP → A verifies via OTP JWT (issuer JWKS)
```

## 7. Two-Token Authorization Model (units-api)

```mermaid
flowchart TB
    REQ["{context, payload} request"] --> SA["AuthenticateServiceAccount<br/>context.developerToken = base64(clientId:secret)<br/>→ central registry resolve (fail-closed cache)"]
    SA --> SC["CheckServiceAccountScope<br/>api.id → scope catalog → SA scopes<br/>+ allowed_operations dot-path narrowing"]
    SC --> RL[RateLimit — registry tier/rules]
    RL --> UA["AuthenticateUser (optional-if-absent)<br/>context.authorization = user JWT"]
    UA --> US["CheckUserScope (skipped if no user)"]
    US --> CO["RequireConsent (mint/transact/update/workflows)"]
    CO --> SIG["RequireSignature (transact only):<br/>JWS over JCS-canonical payload,<br/>key from key_references"]
    SIG --> OPA["OPA/Rego at resource level:<br/>Rule 1 owner · Rule 2 class manager ·<br/>Rule 2b any identity may VIEW · Rule 3 delegation"]
```

## 8. Chain Access Path

```mermaid
sequenceDiagram
    participant A as units-api
    participant O as adapterOrchestrator
    participant DB as adapter_registry
    participant C as alchemyAdapter
    participant W as Workers (Broadcast/Confirm/SafetyNet/Expiry)
    participant K as Kafka (finternet.transactions.*)
    participant R as Chain RPC

    A->>O: POST /api/v1/chain-adapter/... {context.chainId}
    O->>DB: chain_ids @> [chainId] AND active, priority DESC
    O->>C: same path, body verbatim
    C->>R: JSON-RPC (EVM) / Solana RPC
    Note over C: Build → intent 'built' (TTL 5m EVM / 60s Sol)
    C->>K: Submit(signedTx) → 'queued' → submit topic
    K->>W: BroadcastWorker → broadcast → 'submitted' → confirm topic
    K->>W: ConfirmationWorker self-requeues (2–5s, ≤720 attempts)
    W->>R: receipt poll → 'confirmed' / 'failed' → status topic / DLQ
```

## 9. MPC Wallet Setup & Signing

```mermaid
sequenceDiagram
    participant B as Browser (walletprovider-sdk)
    participant X as BFF wallet module
    participant W as wpbe-silencelabs
    participant SN as Silent Network nodes
    participant A as units-api

    B->>X: WS /v1/registerPasskey
    X->>W: WS proxy (upgrade)
    W->>SN: WebAuthn ceremony (challenge → attestation)
    B->>X: POST /api/v1/wallet/create
    X->>W: POST /v2/rest/keygen
    W->>SN: threshold DKG (keyshares stay on SN nodes)
    W-->>X: key_id, address (ETH from secp256k1 / base58 from ed25519), public_key
    X->>X: persist user_passkeys + user_mpc_keys (Prisma)
    X->>A: POST /v1/account/keys/register (key_references)
    B->>X: POST /api/v1/wallet/sign-transfer
    X->>W: POST /v2/rest/signgen (chain-validated)
    W->>SN: threshold signing (key never reconstructed)
    W-->>X: signature (ETH: r‖s‖v · Sol: raw)
```

## 10. Proof Generation

```mermaid
flowchart LR
    T[(transactions<br/>proof_id IS NULL,<br/>status completed)] -->|"batch = exactly MERKLE_BATCH_SIZE,<br/>FOR UPDATE"| P[proofService]
    TT[(token_transactions)] --> P
    P -->|"BLAKE3 Merkle tree<br/>leaf = blake3(canonical JSON)"| PR[(proofs<br/>proof_data {root, per-leaf paths}<br/>status 'proven')]
    P -->|stamp proof_id| T & TT
    API["units-api<br/>/v1/transaction/proof · /leaf · /verify"] --> PR
    ANCHOR["anchoring (proven → anchored):<br/>NOT IMPLEMENTED — ledger_anchors always NULL"]:::warn
    classDef warn stroke-dasharray: 5 5
```

## 11. Database ER Overview

```mermaid
erDiagram
    accounts ||--o{ key_references : "keys (mpc/vault/external)"
    accounts ||--o{ user_consents : ""
    terms_versions ||--o{ user_consents : "only enforced FK"
    token_classes ||--o{ tokens : ""
    token_classes ||--|| token_class_configs : "program + hooks"
    token_programs ||--o{ token_class_configs : ""
    tokens ||--o{ token_transactions : "debit/credit entries"
    tokens ||--o{ state_history : "commitment chain"
    transactions ||--o{ token_transactions : ""
    transactions ||--o{ workflow_ops : "primitive audit"
    transactions ||--o{ primitive_ops : "saga lifecycle (6 writers)"
    proofs ||--o{ transactions : "batched"
    workflow_registry ||--o{ workflows : ""
```

Separate databases: **central registry** (accounts-directory, instances, developer credentials, delegations — 12 tables) · **BFF Prisma** (oidc_clients, provider_journeys, integration_providers, user_wallet_links, user_passkeys, user_mpc_keys) · **alchemyAdapter** (chain_adapter_intents, chain_adapter_transactions). Full column detail: [06-database-schemas.md](06-database-schemas.md).

## 12. primitive_ops — the Saga Backbone (writer ownership)

```mermaid
flowchart LR
    subgraph row ["one row per (txn_id, op_seq)"]
        BA["base: method, status, details, result"]
        A["(A) admit: request_hash, admit_*"]
        Bg["(B) command outbox → Kafka"]
        Cg["(C) engine dedup + result"]
        Dg["(D) completion outbox → source"]
        Eg["(E) signal outbox → Restate"]
    end
    GW["gateway admit (Go)"] --> A & Bg
    CP["command poller (Go)"] --> Bg
    EN["tokenEngine (Rust)<br/>writes C+D inside the state tx"] --> Cg & Dg
    OP["completion poller (Rust)"] --> Dg
    CH["completion handler (Go)"] --> Eg
    SP["signal poller (Go, SOURCE only)"] --> Eg
```

Every writer uses targeted `UPDATE … WHERE (txn_id, op_seq)` on its own column group — never full-row upserts. Three partial indexes let each poller scan only its own pending rows.
