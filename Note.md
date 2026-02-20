```
Interview me in detail using the AskUserQuestionTool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. but make sure the questions are not obvious. Be very in-depth and continue interviewing me continually until it's complete, then write the spec to the file
```

```
naveen.hiremath@finternetlab.io
```

```
lakshmi@example.co.in
```


error handling issue:
* currently when we make external calls (otp-service), even though we have error 400 from otp-service, units its taking as 503 or 500 in generic
* verify end to end errors








Audit the entire Workflow API codebase for `panic()` calls, `log.Fatal()` in request handlers, and unhandled error paths. Replace them with proper `AppError` returns using the structured error types in `workflow/src/errors/app_error.go` (e.g., `apperrors.Internal()`, `apperrors.InvalidInput()`). The recover middleware in `workflow/src/middlewares/recover.go` catches panics but this should be a safety net, not the primary error handling mechanism.

**Acceptance Criteria:**
- [ ] Zero `panic()` calls in controller and service layers
- [ ] All error paths return typed `AppError` with appropriate HTTP status codes
- [ ] Recover middleware only triggers for truly unexpected panics
- [ ] Error responses follow the standard envelope format with error codes

and for otp-service responses as well, whatever error and error status from otp service is currently considered as 503 service unavailable, so its difficult to recognize some errors























* explore on kin (schema validator lib) whether it supports custom functions that can be executed (fun userExist)
* if not we can use our own approach like add custom attribute in json schema so that have middleware which checks all the attributes and if custom attribute is present then checks for address

```
eg.

/token/transact
/parseEnvelope
/validateUserToken
/validateDevToken
/validateRequest(schemaKey)
/hash(fieldName, schemaKey) -> using (kin/openai3)(use better route name) which checks format key from spec and execute util function to check address, fieldName can be like payload.id or payload.identities[].id (use npm `jq` lib), transform it (i.e convert address to addressHash), updated the request value by finding from format key and update it in request
/controller
..
..
..
```
DefineStringFormatCallback
subagents


example-skills
github
playground, playwrote
AskUserQuestionTool
ralph
!

