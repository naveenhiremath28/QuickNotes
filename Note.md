
currently units-api expects signed payload for account create (it may signed from MPC or any other custom logic like using 25519 algorithm), consider if this signature is missed its taking legacy account creation flow, so instead, we need to hit registry resolve and get the signed payload back and continue the federation flow and also we need to create transit/sign key (currently we are only creating transit key where we used for decryption/encryption), since we are also creating transit/sign key we can use it for signing in further like while transfer

1. Add resolve flow in UNITS 
	1. Add signature validation middleware 
	2. Add vault key creation to account creation flow 
	3. Validate signed registry responses


currently units-api expects `signedRegisterNameEnvelope` in payload for account create (it may signed from MPC or any other custom logic like using 25519 algorithm).  consider if this `signedRegisterNameEnvelope` is missed its taking legacy account creation flow, so instead we need to hit registry and for trust, registry can sign the response from it private key and units-api verifies it with registry's public key and continue the federation flow and also we need to create transit/sign key (currently we are only creating transit key where we used for decryption/encryption), since we are also creating transit/sign key we can use it for signing in further like while transfer

note that we cannot use new transit/sign for signing it we need to hit registry and get the availability signed response which is enough and can continue the federation account creation flow  