
1. Throwing error if address contains invalid characters; only alphanumeric, hyphens, underscores, and dots are allowed - Expected
2. logging address 
```log
[restate][2026-06-17T09:54:05.385Z][signup/019ed500-91f0-702e-bf94-04ad104f909d/run][inv_1bv3kxEDSIVY01BauAjLUVwLTc3PAXeIMy] INFO: Starting invocation.

prisma:warn Prisma failed to detect the libssl/openssl version to use, and may not work as expected. Defaulting to "openssl-1.1.x".
Please manually install OpenSSL and try installing Prisma again.

2026-06-17T09:54:05.450Z INFO signup_workflow_started {"txn_id":"019ed500-91f0-702e-bf94-04ad104f909d","account_id":"019ed500-91f0-702e-bf94-04ad104f909d","did":"did:units:0x3039a76bbb2ce365e717ec24f806fe192eec7020ae9e069c823ea9b4d677bc86","home_instance":"a.local"}

2026-06-17T09:54:05.490Z INFO signup_register_name_committed {"txn_id":"019ed500-91f0-702e-bf94-04ad104f909d","account_id":"019ed500-91f0-702e-bf94-04ad104f909d","did":"did:units:0x3039a76bbb2ce365e717ec24f806fe192eec7020ae9e069c823ea9b4d677bc86","response":{"address":"naveen.hiremath","did":"did:units:0x3039a76bbb2ce365e717ec24f806fe192eec7020ae9e069c823ea9b4d677bc86","home":"a.local","home_endpoints":{"api_url":"http://units-api-1:3000","ulip_url":"http://units-api-1:3000"},"email_hash":"f0b2e5a018fbb5b6b11ade7563d14fc2193e2a8f7d19ba7b37e486d0194a7596","version":1,"as_of":"2026-06-17T09:54:05.483Z","signed_by":"registrar.finternet.lab","key_id":"registrar-key-1","signature":"2EuMOSgBdu/rBvcCbabwubo5hgAy7Jt6KGtTGrNkwh81WTwnRrPVbHbSV+QtmHKd8arRlGMCl5Gk3uQTZXJ6BQ=="}}

2026-06-17T09:54:05.507Z INFO signup_finalize_committed {"txn_id":"019ed500-91f0-702e-bf94-04ad104f909d","account_id":"019ed500-91f0-702e-bf94-04ad104f909d","status":"COMMITTED"}

2026-06-17T09:54:05.518Z INFO signup_workflow_completed {"txn_id":"019ed500-91f0-702e-bf94-04ad104f909d","account_id":"019ed500-91f0-702e-bf94-04ad104f909d","did":"did:units:0x3039a76bbb2ce365e717ec24f806fe192eec7020ae9e069c823ea9b4d677bc86"}

[restate][2026-06-17T09:54:05.520Z][signup/019ed500-91f0-702e-bf94-04ad104f909d/run][inv_1bv3kxEDSIVY01BauAjLUVwLTc3PAXeIMy] INFO: Invocation completed successfully.
```


3. **Failing** :-
If the user hits instance-2 but their account is on instance-1: 
```json 
{ 
	"code": "FORWARD", 
	"home_instance": "a.local", 
	"api_url": "http://units-api-1:3000", 
	"message": "Account is hosted on another instance" 
} 
``` 
The client retries the login at the correct instance.

4. transfer is failing due to schema has not been updated
5. support transfer in app
6. Verified Scenario

```

  ---
  Common Entry Point — OTP Verification at Registry

  (Shared by registry-based signup and login. No workflow here.)

  Step 1 — Request OTP
  - User → Registry: POST /v1/account/login (username only).
  - Registry → OTP service: POST /api/v1/otp/generate.
  - Registry → User: "OTP sent."

  Step 2 — Verify OTP
  - User → Registry: POST /v1/account/login (username + OTP).
  - Registry → OTP service: POST /api/v1/otp/verify → returns OTP JWT.
  - Registry: checks if username exists in its name catalog.
  - Registry → User:
    - Existing → { is_existing: true, home_instance, did, access_token } → Flow B.
    - New → { is_existing: false, instances:[...] } → Flow A.

  ▎ ⚙️ No workflow triggered. OTP send/verify and existence check are plain synchronous calls.

  ---
  Flow A — Signup: OTP at Registry → Account Create at Instance

  1. User picks a home instance.
  2. User → Instance: POST /v1/account/create (OTP JWT + dev token, account details).
  3. Instance: validates OTP JWT.
  4. Instance → Registry: POST /v1/account/create — reserves name; Registry returns signed reservation proof.
  5. Instance: verifies reservation-proof signature (fail-closed).
  6. Instance: confirms no local duplicate; encrypts PII.
  7. Instance → Vault: provisions Ed25519 signing key (Vault Transit).
  8. Instance: derives DID from Vault pubkey; writes account + key reference.
  9. Instance → Keycloak: creates and links auth user.

  10. ⚙️ WORKFLOW TRIGGERED HERE — after the local account, key, and Keycloak user exist, the Instance triggers the Restate signup workflow (run synchronously; workflow id = account id). The workflow
  performs:
    - a. signup_register_name_committed → Instance → Registry POST /ulip/v1/name/register — commits the name→DID binding; Registry signs it as registrar.
    - b. signup_finalize_committed → workflow calls back Instance POST /v1/internal/accounts/finalize to mark the account COMMITTED.
    - c. signup_workflow_completed.
  11. Instance → User: 200 { accountId, did, status: "active" }.

  ▎ ⚙️ Workflow: signup, triggered at step 9, synchronous (no client polling). Owns name registration + finalize. The account/key/Keycloak setup happens before the workflow; name commit + finalize happen
  ▎ inside it.

  ---
  Flow B — Login: OTP at Registry → Token Exchange at Instance

  12. User → Instance: POST /v1/account/login (ssoLogin: true, Bearer OTP JWT).
  13. Instance: extracts bearer JWT.
  14. Instance → Registry: GET /.well-known/ulip.json — fetches Registry pubkey.
  15. Instance: verifies signature; reads username from verified claims only.
  16. Instance: looks up local account.
  17. Instance → Keycloak: token exchange → local instance JWT.
  18. Instance → User: { access_token, ... }.

  ▎ ⚙️ No workflow triggered. Pure synchronous JWT verification + token exchange.

  ---
  Flow C — Direct Login / Signup via Instance (no Registry OTP step)

  C1 — Direct Signup
  19. User → Instance: POST /v1/account/create.
  20. Instance: validates; confirms no local duplicate; encrypts PII.
  21. Instance → Vault: provisions Ed25519 key; derives DID.
  22. Instance: writes account + key reference.
  23. Instance → Keycloak: creates and links auth user.

  24. ⚙️ WORKFLOW TRIGGERED HERE — same as Flow A: the Instance triggers the signup workflow to commit the name binding (Instance → Registry POST /ulip/v1/name/register) and run finalize. (Name uniqueness
  lives at the Registry regardless of entry path, so the workflow still runs.)
  25. Instance → User: 200 { accountId, did, status: "active" }.

  C2 — Direct Login
  26. User → Instance: POST /v1/account/login (username + credential, no ssoLogin).
  27. Instance: authenticates locally; looks up account.
  28. Instance → Keycloak: token exchange → local instance JWT.
  29. Instance → User: { access_token, ... }.

  ▎ ⚙️ C1: triggers the signup workflow (same as Flow A). C2: no workflow — login never runs one.

  Rule of thumb: the Restate signup workflow fires only on account creation, triggered by the Instance near the end of the create handler, and runs synchronously. Every login path (B and C2) and the
  OTP/existence-check path at the Registry trigger no workflow.
```




