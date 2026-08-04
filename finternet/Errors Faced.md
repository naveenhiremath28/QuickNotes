### Account signup flow:

1. OTP_SERVICE_URL missing in registry, taking default url https://otp-service:4000 and getting 500 → manually resolved
2. Added ingress in registry for admin url which maps to :3001 internally and mapped to REGISTRY_ADMIN_URL= in units-api → Manually resolved
3. Unable to register during instance startup in registered_instance table, due to missing REGISTRY_BASE_URL in units-in-sanctum → manually resolved
4. address/availability api → cloudflare issue -> resolved
5. Instances are entering `https://units.sanctum-sg.finternetlab.io/v1` , for ulip - `https://units.sanctum-sg.finternetlab.io/ulip` instead  `https://units.sanctum-sg.finternetlab.io` registered_instances, due to wrong `PUBLIC_API_URL` and `PUBLIC_ULIP_URL` - Manually resolved
6. private key (otp-service) should match public key (same pub key for all units-api instances) → Manually resolved
7.  It's a misleading error message, not a duplicate. Recall the code (account.go:427-433): - Not fixed yet

```go
if wfResult.Status != "COMMITTED" {
    service.rollbackFederationAccount(ctx, log, accountID)
    log.Warn("registry_registration_rejected", ... status, reason)
    return nil, apperrors.AlreadyExists("account already registered in the federation; sign in instead", nil)
}
```

8. signup workflow not deployed at all — missing from the ArgoCD restate-workflows-orchestrator workflows: list, so no signup pod existed → Restate returned 404 service 'signup' not found → account create 503. Added the signup entry → manually resolved
9. CRYPTO_CONFIG.token = "OVERRIDE_ME" (placeholder never overridden) → the TransitSigningService client sent an invalid token → Vault 403 permission denied / invalid token on transit/keys/units-signing-* → account create 500 (and orphaned account rows caused misleading 409s) → manually resolved
10. ULIP_SIGNING_PRIVATE_KEY missing on the signup workflow pod → getSigner() returned null → RegisterName ULIP envelope sent unsigned (no signature object) → registry 400 "context.method, context.caller_instance and signature.signer_instance are required" → manually resolved

### Token Mint:

1.  Stale published capability — units-in's registered_instances.capability had an old as_of snapshot missing the active token programs → token mint 400 primitive_capability_missing - Not fixed yet
2. decryption was blocked by vault - Manually resolved
3. In restate workflow orchestrator ULIP_BASE_URL was configured registry url instead units-in - Manually resolved
4. For token transactions, transactions DDL defined units instead token-engine

### Token Transaction

1. otp-service need to expose for units instances currently exposed only for registry (failed during token transaction when units api try to use otp-service)- Manually resolved
2. otp-service - added new routes
3. disabled rewrite annotation
4. data seed for scopes and scope_apis -> Manually seeded





---

remove -   targetUrlAccount: "https://units.sanctum-sg.finternetlab.io" 
