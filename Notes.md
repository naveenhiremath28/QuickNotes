1. In `key_references` table  
	1. `address` field but storing did value
	2. remove user_public_key field, no longer required
2. In `registered_names` table 
	1. missing values for the fields - did, email/phone hash, user_public_key, since email/phone hash are missed unique check from registry is failing
3. In `accounts` table
	1. missing values for the fields - email/mobile, pii
4. `Instance POST /v1/account/create` api, according to new spec returning as expected response
```json
   "response": {
	"accountId": "019ecb4c-0629-73dc-ac4d-a974790a9c12",
	"did": "did:units:0xaed0900785f4....",
	"status": "active"
	}
```

	app expects keycloak's jwt in response, or else app has to hit login api again to get keycloak's jwt 
