

**OTP Service (Node.js)**

1. **Structured Error System Introduced**

File: otp-service/src/utils/errors.js  
- Added ErrorCodes enum (OTP_EXPIRED, OTP_INVALID, TWILIO_*, JWT_GENERATION_FAILED, etc.).  
- Added code field to AppError.

2. **Controllers Updated**

File: otp-service/src/controllers/otp.controller.js  
- Replaced direct res.status().json() responses with typed AppError.  
- JWT failures now throw JWT_GENERATION_FAILED instead of generic errors.

3. **JWT Service Hardened**

File: otp-service/src/services/jwt.service.js  
- Replaced new Error() with AppError / ValidationError.  
- Added proper error codes such as CONFIGURATION_ERROR and VALIDATION_FAILED.

4. **Twilio Error Handling Centralized**

File: otp-service/src/services/twilio.service.js  
- Added mapTwilioError() function.  
- Mapped Twilio errors properly:
  - Rate limit → 429  
  - Invalid recipient → 400  
  - Service unavailable → 503  
- Wrapped send and verify methods in try/catch blocks.

5. **Error Middleware Improved**

File: otp-service/src/middleware/errorHandler.js  
- Added error_code and retry_after in responses.  
- Removed hardcoded Twilio error handling logic.


---------------------------------------------------------------------

**Workflow API (Go)**

1. **OTP-Specific Error Constructors Added**

File: workflow/src/errors/app_error.go  
- Added five new constructors:
  - OTPExpired() (401)  
  - OTPInvalid() (401)  
  - OTPRateLimited() (429)  
  - OTPBadRequest() (400)  
  - OTPTimeout() (503)

2. **OTP Error Mapping Between Services**

File: workflow/src/services/otp.go  
- Added handleOTPServiceError().  
- Added:
  - mapOTPErrorCode() (primary mapping)  
  - mapOTPStatusCode() (fallback mapping)  
- Ensures consistent OTP error handling between services.

3. **Panics Removed (Crash Prevention)**

File: workflow/src/controllers/account.go  
- Replaced panic() with apperrors.Internal().

File: workflow/src/config/config.go  
- Replaced mustJSON() (which panicked) with marshalJSON() (returns error).

File: workflow/src/providers/credential/registry.go  
- Duplicate transformer registration now returns an error instead of panicking.

File: workflow/src/providers/credential/signzy/init.go  
- Updated to properly handle the returned error.

4. Infrastructure Error Masking (Security)

File: workflow/src/providers/crypto/vault/provider.go  
- Wrapped Vault errors in safe application errors.  
- Added security documentation comments explaining intentional re-panics.

File: workflow/src/services/kafka.go  
- Wrapped Kafka configuration and publish errors in application errors.  
- Prevented exposure of internal broker details to clients.

5. Silent Type Assertion Fixes

**Files:**
- workflow/src/services/token_class.go  
- workflow/src/services/transaction.go  
- workflow/src/utils/token_transaction_helpers.go  

- Replaced silent type assertion failures with warning logs.

6. Panic Logging Improved

File: workflow/src/middlewares/recover.go  
- Enhanced logging to include:
  - panic.value  
  - http.method  
  - http.path  
  - correlation_id

7. Constants Added

File: workflow/src/constants/messages.go  
- Added MsgOTPServiceUnavailable.

---------------------------------------------------------------------

**Final Impact**

- Three major panics removed  
- Sensitive internal details hidden from clients  
- Structured error codes implemented across both services  
- Proper OTP error mapping between services  
- Silent failures converted to warning logs  
- Tests updated to verify secure error behavior 