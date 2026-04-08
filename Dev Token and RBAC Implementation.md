
# Developer Token Flow & Schema

## Database Tables

### `api_clients`

```sql

CREATE TABLE api_clients (  
    id                  UUID PRIMARY KEY,        -- UUIDv7  
    account_id          UUID NOT NULL,           -- → accounts  
    name                VARCHAR(255) NOT NULL,  
    description         TEXT,  
    scopes              JSONB DEFAULT '[]',      -- ["tokens:view", "tokens:transact"]  
    allowed_operations  JSONB DEFAULT '{}',      -- {"tokens:transact": ["transfer","split"]}  
    rate_limit_tier_id  UUID,                    -- → rate_limit_tiers (optional)  
    status              VARCHAR(20) DEFAULT 'active',  
    created_at          TIMESTAMPTZ,  
    updated_at          TIMESTAMPTZ  
);
```

---

### `api_keys`

```sql
CREATE TABLE api_keys (  
    id            UUID PRIMARY KEY,  
    client_id     UUID NOT NULL,                 -- → api_clients  
    key_hash      VARCHAR(64) UNIQUE NOT NULL,   -- SHA-256 hex (64 chars)  
    name          VARCHAR(255) NOT NULL,  
    status        VARCHAR(20) DEFAULT 'active',  
    expires_at    TIMESTAMPTZ NOT NULL,          -- default 90 days  
    last_used_at  TIMESTAMPTZ,  
    revoked_at    TIMESTAMPTZ,  
    created_at    TIMESTAMPTZ,  
    updated_at    TIMESTAMPTZ  
);
```

---

## Relationships

```
accounts (1) → (N) api_clients (1) → (N) api_keys  
                        ↓ optional  
                 rate_limit_tiers
```

- Max **10 clients per account**
- Max **5 active keys per client**

---

## API Key Generation Flow

1. Generate 32 random bytes using `crypto/rand`
2. Encode as **base62** (alphanumeric, no special characters)
3. Prepend prefix → `fnt_ABC123...` (~47 chars total)
4. Hash using **SHA-256**
5. Store only the **hash** in DB
6. Return plaintext **only once**

---

## Authentication Middleware Flow

ParseEnvelope  
  → AuthenticateDeveloper  
  → CheckClientScope  
  → RateLimit  
  → Authenticate (JWT, optional)  
  → ValidateRequest  
  → Handler

### AuthenticateDeveloper (Step-by-step)

1. **Feature flag check**
    - If `enableDevTokenVerification = false` → skip (bootstrap mode)
2. **Extract token**
    - From `context.developer_token` OR `Authorization` header
3. **Format validation**
    - Must start with `fnt_`
    - Length: 40–60 characters
4. **Hash & DB lookup**

SELECT key_id, client_id, account_id, scopes, allowed_operations, rate_limit_tier_id  
FROM api_keys  
JOIN api_clients ON ...  
WHERE key_hash = ?  
  AND status = 'active'  
  AND expires_at > NOW()  
  AND client.status = 'active';

5. **On success**
    - Store in context:
        - `account_id`
        - `client_id`
        - `scopes`
        - `allowed_operations`
6. **On failure**
    - `401` → not found / expired / revoked
    - `503` → DB error (fail-closed)
7. **Usage tracking**
    - `usageTracker.Track(keyID)` (batched)

---

## JWT Behavior

- ❌ No JWT → use dev token's account
- ✅ Valid JWT → overrides dev token account
- ❌ Invalid JWT → request rejected (`401`)

---

## API Endpoints (`/v1/clients/*`)

|Method|Path|Scope|Description|
|---|---|---|---|
|POST|`/clients/register`|`clients:create`|Create new API client (max 10/account)|
|POST|`/clients/get`|`clients:view`|Get client by ID (ownership verified)|
|POST|`/clients/list`|`clients:view`|List all clients|
|POST|`/clients/update`|`clients:manage`|Update name/description|
|POST|`/clients/deactivate`|`clients:create`|Soft delete client + revoke all keys|
|POST|`/clients/keys/create`|`keys:create`|Generate new key (plaintext returned once)|
|POST|`/clients/keys/list`|`keys:view`|List key metadata (no secrets)|
|POST|`/clients/keys/revoke`|`keys:manage`|Revoke key immediately|

---

## Usage Tracker

- In-memory map: `keyID → lastUsedTimestamp`
- Flush to DB every **60 seconds (batch)**
- Retry up to **3 times**, then drop
- On shutdown → **final flush**

---

## Bootstrap Flow (First-Time Setup)

1. Set `enableDevTokenVerification = false`
2. Start service
3. Create account → `/account/create`
4. Register client → `/clients/register`
5. Generate key → `/clients/keys/create`
6. Save plaintext key ⚠️
7. Set `enableDevTokenVerification = true`
8. Restart service

---

## Security Summary

|Property|Implementation|
|---|---|
|No plaintext storage|Only SHA-256 hash stored|
|Immediate revocation|Direct DB lookup (no cache)|
|Fail-closed|DB errors → `503`|
|No info leakage|Generic `401` responses|
|Ownership isolation|`404` for unauthorized access|
|Feature-flagged|Safe rollout|
