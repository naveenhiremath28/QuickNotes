# Token engine startup flow
_Exported on 1/23/2026 at 12:04:47 GMT+5:30 from Cur
sor (2.3.41)_

---

**Cursor**


Reviewing the token engine's initialization and startup flow.



Reviewing the operation executor and program registry to understand the execution flow:



## Token Engine Startup Flow

### 1. Initialization (`main.rs`)

When the Token Engine starts, it follows this sequence:

```
1. Load Configuration
   ↓
2. Initialize Telemetry (OpenTelemetry)
   ↓
3. Connect to Database (PostgreSQL)
   ↓
4. Initialize Program Registry
   ↓
5. Create Kafka Producer (for audit events & DLQ)
   ↓
6. Create Operation Executor
   ↓
7. Start Health Server (Kubernetes probes)
   ↓
8. Create Kafka Consumer
   ↓
9. Start Consumer Loop (background task)
   ↓
10. Wait for Shutdown Signal
```

### 2. Program Registry Initialization

The `ProgramRegistry` is created and automatically registers:

**Default Programs:**
- `reference-ft` - Reference Fungible Token program
- `reference-nft` - Reference NFT program

**Default Hooks:**
- `logging` - Logs operations
- `validation` - Validates operations
- `max-supply` - Enforces max supply on mint
- `min-balance` - Enforces minimum balance on transfers/burns

These are loaded from the `tokenPrograms` workspace at compile time.

### 3. Kafka Consumer Loop

The consumer runs in a background task and continuously:

```rust
loop {
    1. Receive message from Kafka topic (units.token.operations)
    2. Parse TokenOperationMessage
    3. Extract trace context (for distributed tracing)
    4. Process message → Execute operation
    5. Commit offset on success
    6. Handle failures (retry or send to DLQ)
}
```

### 4. Operation Execution Flow

When a message is received, here's the detailed execution:

```
┌─────────────────────────────────────────────────────────┐
│ 1. Parse Message                                        │
│    - Extract tx_id, operation, token_class, payload     │
│    - Extract trace context for distributed tracing     │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 2. Update Transaction Status                            │
│    - Set status to "processing"                         │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 3. Load Token Class Configuration                       │
│    - Get program_id from database                       │
│    - Get token_standard (FT/NFT)                        │
│    - Get token metadata (maxSupply, decimals, etc.)     │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 4. Resolve Token Program                                │
│    - Lookup program from registry by program_id         │
│    - Verify program supports token_standard             │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 5. Get or Create Token State                           │
│    - For mint: create new or find existing token       │
│    - For transfer: fetch sender's token                 │
│    - For other ops: fetch by token_id                   │
│    - Load state history (previous_commitment)          │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 6. Verify State Commitment (Mandatory Pre-Hook)        │
│    - Verify existing token hasn't been tampered with    │
│    - Uses historical commitment_config                  │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 7. Validate Operation                                   │
│    - Program validates operation is supported          │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 8. Run Pre-Hooks (from config)                         │
│    - Validation, MaxSupply, MinBalance, etc.             │
│    - Ordered by priority                                │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 9. Execute Token Program                                │
│    - program.execute(ctx, operation, token_state)      │
│    - Returns: OperationResult with new_state           │
│    - May return multiple affected_states (transfers)    │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 10. Persist State Changes (Atomic Transaction)         │
│     - Insert/update tokens                               │
│     - Create token_transactions records                 │
│     - Create state_history entries                      │
│     - All in single DB transaction                      │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 11. Run Post-Hooks (from config)                        │
│     - Logging, custom hooks, etc.                       │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 12. Audit Logging (Mandatory Post-Hook)                 │
│     - Record operation in audit_events table            │
│     - Optionally publish to Kafka audit topic           │
└──────────────────┬──────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────────────────────┐
│ 13. Update Transaction Status                           │
│     - Set to "completed" or "failed"                    │
│     - Store collected identities                        │
└─────────────────────────────────────────────────────────┘
```

### 5. Token Program Execution

When `program.execute()` is called:

1. The program receives:
   - `ExecutionContext` - tx_id, correlation_id, initiator, token_class_data
   - `Operation` - operation type, token_id, payload
   - `TokenState` - current token state

2. The program:
   - Validates the operation
   - Computes new state (e.g., updates balance, changes owner)
   - Computes new `state_commitment` (cryptographic hash)
   - Returns `OperationResult` with new state

3. For transfers (fungible tokens):
   - Returns multiple `affected_states`:
     - Debit entry (sender's token)
     - Credit entry (recipient's token)

### 6. Key Components

**OperationExecutor:**
- Orchestrates the full lifecycle
- Manages database transactions
- Runs hooks
- Handles state persistence

**ProgramRegistry:**
- In-memory registry of available programs
- Lookup by program_id
- Manages hook configurations

**TokenEventConsumer:**
- Kafka consumer loop
- Message processing with retry logic
- DLQ support for failed messages
- Health state management

**Hooks:**
- Pre-hooks: validation before execution
- Post-hooks: logging/notification after execution
- Mandatory hooks: commitment verification, audit logging

### 7. Error Handling

- Retryable errors: transient failures (DB connection, etc.)
- Non-retryable errors: validation failures, program errors
- DLQ: failed messages after max retries are sent to dead-letter queue
- Transaction status: always updated to reflect final state

This architecture separates concerns: the Token Engine handles orchestration, persistence, and hooks, while Token Programs implement the business logic for each operation type.

---
**Cursor**


Tracing the execution flow with code references. Showing where each step occurs:



Reviewing a few more sections to complete the flow:



Execution flow with code references:

## Execution flow with code references

### Step 1: Parse Message
Location: `tokenEngine/src/kafka/consumer.rs:151-214`

```rust:151-165:tokenEngine/src/kafka/consumer.rs
async fn process_message<M: Message>(&self, message: &M) -> Result<(), EngineError> {
    let payload = message.payload().ok_or(EngineError::EmptyPayload)?;

    let event: TokenOperationMessage =
        serde_json::from_slice(payload).map_err(EngineError::Serialization)?;

    // Check if we have a valid parent trace context
    let has_parent = is_valid_trace_context(&event.trace_context);
    let parent_context = extract_parent_context(&event.trace_context);

    let tx_id = event.tx_id;
    let operation = format!("{:?}", event.operation);
    let token_class = event.token_class.clone();
    let correlation_id = event.correlation_id.clone();
```

Extracts:
- `tx_id` - Transaction ID
- `operation` - Operation type (Mint, Transfer, etc.)
- `token_class` - Token class identifier
- `payload` - Operation-specific data
- `trace_context` - Distributed tracing context

Then calls executor:
```rust:206-210:tokenEngine/src/kafka/consumer.rs
with_retry(&retry_config, &format!("execute:{}", tx_id), || {
    let executor = executor.clone();
    let event = event.clone();
    async move { executor.execute(event).await }
})
```

---

### Step 2: Update Transaction Status
Location: `tokenEngine/src/engine/executor.rs:104-116`

```rust:104-116:tokenEngine/src/engine/executor.rs
// Upsert transaction status to processing
// This will INSERT if transaction doesn't exist (Token Engine creates it)
// or UPDATE if it already exists (Workflow API created it)
self.transaction_repo
    .upsert_status(
        tx_id,
        &message.correlation_id,
        &message.initiator,
        TransactionStatus::Processing.as_str(),
        None,
        None, // No identities yet
    )
    .await?;
```

Sets transaction status to "processing" before execution.

---

### Step 3: Load Token Class Configuration
Location: `tokenEngine/src/engine/executor.rs:163-211`

```rust:163-173:tokenEngine/src/engine/executor.rs
// Get token class config
let config = self
    .config_repo
    .get_by_token_class(&message.token_class)
    .await?
    .ok_or_else(|| EngineError::ConfigNotFound {
        token_class: message.token_class.clone(),
    })?;

// Check if this token class is fungible (determines transfer behavior)
let is_fungible = self.config_repo.is_token_fungible(&message.token_class).await?;
```

```rust:184-211:tokenEngine/src/engine/executor.rs
// Get the token_standard for this token class
let token_standard = self.config_repo.get_token_standard(&message.token_class).await?;

// [PRE-VALIDATION] Verify program supports the token standard
let supported_standards = program.supported_standards();
if !supported_standards.contains(&token_standard) {
    return Err(EngineError::UnsupportedTokenStandard {
        token_standard: token_standard.clone(),
        program_id: program_id.clone(),
        supported: supported_standards,
    });
}

// Fetch token class metadata (contains constraints like maxSupply, minBalance, decimals, etc.)
let token_class_metadata = self
    .config_repo
    .get_token_class_metadata(&message.token_class)
    .await?;
```

Loads:
- `program_id` - Which token program to use
- `token_standard` - FT or NFT
- `is_fungible` - Whether tokens are fungible
- `token_class_metadata` - maxSupply, decimals, etc.

---

### Step 4: Resolve Token Program
Location: `tokenEngine/src/engine/executor.rs:175-195`

```rust:175-195:tokenEngine/src/engine/executor.rs
// Get the token program
let program_id = &config.program_id;
let program = self
    .registry
    .get_program(program_id)
    .map_err(|_| EngineError::ProgramNotFound {
        program_id: program_id.clone(),
    })?;

// Get the token_standard for this token class
let token_standard = self.config_repo.get_token_standard(&message.token_class).await?;

// [PRE-VALIDATION] Verify program supports the token standard
let supported_standards = program.supported_standards();
if !supported_standards.contains(&token_standard) {
    return Err(EngineError::UnsupportedTokenStandard {
        token_standard: token_standard.clone(),
        program_id: program_id.clone(),
        supported: supported_standards,
    });
}
```

Looks up the program from the registry and verifies it supports the token standard.

---

### Step 5: Get or Create Token State
Location: `tokenEngine/src/engine/executor.rs:350-352` and `529-646`

```rust:350-353:tokenEngine/src/engine/executor.rs
// Get or create token state (also returns commitment_config for verification)
let (token_state, is_new_token, historical_commitment_config) =
    self.get_or_create_token_state(message, &config, is_fungible).await?;
let old_state = token_state.clone();
```

The `get_or_create_token_state` method handles different cases:

For Mint operations:
```rust:538-560:tokenEngine/src/engine/executor.rs
// For mint operations, check if we're adding to existing or creating new
if matches!(operation_type, OperationType::Mint) {
    // For fungible tokens, check by owner + token_class
    if is_fungible {
        // Extract owner from payload (Issuer in identities becomes Owner)
        let owner = self.extract_owner_from_mint_payload(&message.payload)?;

        // Check if owner already has a token of this class
        if let Some(token_model) = self
            .token_repo
            .find_by_owner_and_class(&owner, &message.token_class)
            .await?
        {
            let mut token_state = StateManager::model_to_state(&token_model)?;
            // Load state history data (previous_commitment + commitment_config)
            let commitment_config = self.load_state_history_data(&mut token_state).await?;
            return Ok((token_state, false, commitment_config));
        }
```

For Transfer operations:
```rust:614-628:tokenEngine/src/engine/executor.rs
// For fungible token transfers, we need the sender's token
// The sender is determined by the initiator (you can only transfer your own tokens)
if is_fungible && matches!(operation_type, OperationType::Transfer) {
    let sender_address = &message.initiator;

    let sender_token = self
        .token_repo
        .find_by_owner_and_class(sender_address, &message.token_class)
        .await?
        .ok_or_else(|| EngineError::TokenNotFound {
            token_id: format!("{}:{}", sender_address, message.token_class),
        })?;

    let mut token_state = StateManager::model_to_state(&sender_token)?;
    // Load state history data (previous_commitment + commitment_config)
    let commitment_config = self.load_state_history_data(&mut token_state).await?;
    return Ok((token_state, false, commitment_config));
}
```

For other operations:
```rust:631-645:tokenEngine/src/engine/executor.rs
// For other operations, token must exist
let token_id = message.token_id.ok_or(EngineError::MissingTokenId)?;

let token_model = self
    .token_repo
    .get_by_id(token_id)
    .await?
    .ok_or_else(|| EngineError::TokenNotFound {
        token_id: token_id.to_string(),
    })?;

let mut token_state = StateManager::model_to_state(&token_model)?;
// Load state history data (previous_commitment + commitment_config)
let commitment_config = self.load_state_history_data(&mut token_state).await?;
Ok((token_state, false, commitment_config))
```

---

### Step 6: Verify State Commitment (Mandatory Pre-Hook)
Location: `tokenEngine/src/engine/executor.rs:355-359`

```rust:355-359:tokenEngine/src/engine/executor.rs
// [MANDATORY PRE-HOOK] Verify state commitment for existing tokens (tamper detection)
// Uses the historical commitment_config to ensure backward-compatible verification
if !is_new_token {
    self.commitment_hook.verify(&token_state, historical_commitment_config.as_ref())?;
}
```

Verifies the token's state commitment to detect tampering. Uses the historical `commitment_config` for backward compatibility.

---

### Step 7: Validate Operation
Location: `tokenEngine/src/engine/executor.rs:361-362`

```rust:361-362:tokenEngine/src/engine/executor.rs
// Validate operation is supported
program.validate_operation(&operation, &token_state)?;
```

The program validates that the operation is supported for this token state.

---

### Step 8: Run Pre-Hooks (from config)
Location: `tokenEngine/src/engine/executor.rs:364-382`

```rust:364-382:tokenEngine/src/engine/executor.rs
// Get hooks from config
let pre_hooks = config.get_pre_hooks();
let post_hooks = config.get_post_hooks();

info!(
    tx_id = %ctx.tx_id,
    program_id = %program_id,
    pre_hook_count = pre_hooks.len(),
    post_hook_count = post_hooks.len(),
    pre_hooks = ?pre_hooks.iter().map(|h| &h.hook_id).collect::<Vec<_>>(),
    post_hooks = ?post_hooks.iter().map(|h| &h.hook_id).collect::<Vec<_>>(),
    phase = "hooks_resolved",
    "hooks_configuration_loaded"
);

// Run pre-hooks
self.hook_runner
    .run_pre_hooks(&ctx, &operation, &token_state, &pre_hooks)
    .await?;
```

The hook runner executes pre-hooks in priority order:
```rust:24-100:tokenEngine/src/engine/hook_runner.rs
pub async fn run_pre_hooks(
    &self,
    ctx: &ExecutionContext,
    operation: &Operation,
    token_state: &TokenState,
    hook_configs: &[HookConfig],
) -> Result<(), EngineError> {
    // Sort hooks by priority (lower number = higher priority)
    let mut sorted_hooks: Vec<_> = hook_configs.iter().collect();
    sorted_hooks.sort_by_key(|h| h.priority);

    for (index, hook_config) in sorted_hooks.iter().enumerate() {
        // Get the hook from registry
        let hook = self.registry.get_hook(&hook_config.hook_id)?;
        
        // Execute the hook
        let result = hook.pre_execute(ctx, operation, token_state).await?;
        
        // Check outcome (Allow, Block, etc.)
        match result.outcome {
            HookOutcome::Allow => continue,
            HookOutcome::Block => return Err(...),
            // ...
        }
    }
}
```

Common pre-hooks: `validation`, `max-supply`, `min-balance`.

---

### Step 9: Execute Token Program
Location: `tokenEngine/src/engine/executor.rs:384-403`

```rust:384-403:tokenEngine/src/engine/executor.rs
// Execute the operation via Token Program
info!(
    tx_id = %ctx.tx_id,
    program_id = %program_id,
    operation = %operation_type,
    token_id = ?operation.token_id,
    phase = "program_execute",
    "token_program_executing"
);
let result = program.execute(&ctx, operation.clone(), token_state).await?;

info!(
    tx_id = %ctx.tx_id,
    program_id = %program_id,
    token_id = %result.new_state.token_id,
    new_state_version = result.new_state.state_version,
    affected_tokens = result.affected_states.len(),
    phase = "program_complete",
    "token_program_execution_complete"
);
```

Calls the program's `execute` method, which:
- Computes the new token state
- Updates balance/owner/metadata
- Computes the new `state_commitment`
- Returns `OperationResult` with `new_state` and potentially multiple `affected_states` (for transfers)

---

### Step 10: Persist State Changes (Atomic Transaction)
Location: `tokenEngine/src/engine/executor.rs:405-459`

```rust:405-459:tokenEngine/src/engine/executor.rs
// Get commitment config for storing in state_history (for backward-compatible verification)
let commitment_config = config.get_commitment_config();

// Collect all state changes to persist atomically
let mut state_changes: Vec<StateChangeToPersist> = Vec::new();

// Token Program returns all states that need to be persisted (for multi-token operations like transfer)
if result.affected_states.is_empty() {
    // Fallback for backward compatibility - single state
    state_changes.push(StateChangeToPersist {
        old_state: old_state.clone(),
        new_state: result.new_state.clone(),
        is_new_token,
        entry_type: None,
        commitment_config: commitment_config.clone(),
        transfer_sender: None,
    });
} else {
    // For transfers, extract sender from the debit entry
    // Collect each affected state
    for affected in &result.affected_states {
        // ... build state_changes
    }
}

// Persist ALL state changes in a single database transaction (atomic)
self.persist_all_state_changes_atomic(message, &state_changes).await?;
```

The atomic persistence method:
```rust:729-770:tokenEngine/src/engine/executor.rs
/// Persist ALL state changes in a single database transaction.
/// This ensures atomicity - either all changes succeed or none do.
async fn persist_all_state_changes_atomic(
    &self,
    message: &TokenOperationMessage,
    state_changes: &[StateChangeToPersist],
) -> Result<(), EngineError> {
    // Start a database transaction
    let mut tx = self.pool.begin().await.map_err(EngineError::Database)?;

    // Persist each state change within the transaction
    for change in state_changes {
        self.persist_single_state_change_in_tx(
            &mut tx,
            message,
            &change.old_state,
            &change.new_state,
            change.is_new_token,
            change.entry_type.clone(),
            change.commitment_config.clone(),
            change.transfer_sender.clone(),
        )
        .await?;
    }

    // Commit the transaction - all changes succeed together
    tx.commit().await.map_err(EngineError::Database)?;

    Ok(())
}
```

For each state change, it:
- Inserts/updates the `tokens` table
- Creates a `token_transactions` record
- Creates a `state_history` entry

All within a single database transaction.

---

### Step 11: Run Post-Hooks (from config)
Location: `tokenEngine/src/engine/executor.rs:461-474`

```rust:461-474:tokenEngine/src/engine/executor.rs
// Run post-hooks (failures don't roll back)
// For multi-token operations, run post-hooks for each affected token
if result.affected_states.len() > 1 {
    for affected in &result.affected_states {
        let state_before = StateManager::snapshot_to_state(&affected.state_before, &affected.token_state)?;
        self.hook_runner
            .run_post_hooks(&ctx, &operation, &state_before, &affected.token_state, &post_hooks)
            .await?;
    }
} else {
    self.hook_runner
        .run_post_hooks(&ctx, &operation, &old_state, &result.new_state, &post_hooks)
        .await?;
}
```

Post-hooks run after state is committed. Failures are logged but don't roll back.

---

### Step 12: Audit Logging (Mandatory Post-Hook)
Location: `tokenEngine/src/engine/executor.rs:476-505`

```rust:476-505:tokenEngine/src/engine/executor.rs
// [MANDATORY POST-HOOK] Audit logging - record all operations
// This runs after configurable post-hooks to capture the final state
if result.affected_states.len() > 1 {
    // Multi-token operation (e.g., transfer) - log each affected token
    for affected in &result.affected_states {
        let old_state_opt = if affected.is_new {
            None
        } else {
            StateManager::snapshot_to_state(&affected.state_before, &affected.token_state)
                .ok()
        };
        if let Err(e) = self
            .audit_hook
            .log(&ctx, &operation, old_state_opt.as_ref(), &affected.token_state)
            .await
        {
            warn!(error = %e, "audit_logging_failed");
        }
    }
} else {
    // Single token operation
    let old_state_opt = if is_new_token { None } else { Some(&old_state) };
    if let Err(e) = self
        .audit_hook
        .log(&ctx, &operation, old_state_opt, &result.new_state)
        .await
    {
        warn!(error = %e, "audit_logging_failed");
    }
}
```

Records the operation in `audit_events` and optionally publishes to Kafka.

---

### Step 13: Update Transaction Status
Location: `tokenEngine/src/engine/executor.rs:121-152`

```rust:121-152:tokenEngine/src/engine/executor.rs
match result {
    Ok(ref identities) => {
        info!(tx_id = %tx_id, "operation_completed");
        self.transaction_repo
            .upsert_status(
                tx_id,
                &message.correlation_id,
                &message.initiator,
                TransactionStatus::Completed.as_str(),
                None,
                Some(identities.clone()), // Pass collected identities
            )
            .await?;
    }
    Err(ref e) => {
        error!(tx_id = %tx_id, error = %e, "operation_failed");
        let error_json = json!({
            "code": e.error_code(),
            "message": e.to_string(),
        });
        self.transaction_repo
            .upsert_status(
                tx_id,
                &message.correlation_id,
                &message.initiator,
                TransactionStatus::Failed.as_str(),
                Some(error_json),
                None, // No identities on failure
            )
            .await?;
    }
}
```

Updates the transaction status to "completed" or "failed" and stores collected identities.

---

## Summary

The flow moves from:
1. Kafka message parsing → `consumer.rs:151-214`
2. Transaction status update → `executor.rs:104-116`
3. Config loading → `executor.rs:163-211`
4. Program resolution → `executor.rs:175-195`
5. State retrieval → `executor.rs:350-352, 529-646`
6. Commitment verification → `executor.rs:355-359`
7. Operation validation → `executor.rs:361-362`
8. Pre-hooks → `executor.rs:379-382` → `hook_runner.rs:24-170`
9. Program execution → `executor.rs:393`
10. Atomic persistence → `executor.rs:459, 729-770`
11. Post-hooks → `executor.rs:461-474` → `hook_runner.rs:172-254`
12. Audit logging → `executor.rs:476-505`
13. Final status update → `executor.rs:121-152`

All orchestrated by `OperationExecutor` in `tokenEngine/src/engine/executor.rs`.

