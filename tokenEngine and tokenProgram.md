

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