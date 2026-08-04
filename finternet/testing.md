```
Verified flows:

Accounts:
* account creation, login
* account creation, login via google sso
* address already in use - cross instance, same instance

Clients and scopes:
* clients registration along with SA account creation in keycloak
* scope approvals/rejections from super admin
* only approved apis can be used, non approved fails - as expected

Credentials
* signzy flow - credential tokens

Profile
* decrypt api 
* pass keys 
* email update (/account/update)

Tokens and transactions
* mint nfh
* wallet connect, proxy tokens etc
* register keys via publickey(starts with 0xaq.., base64 value), address - using postman
* /keys apis latest changes and camel case fixes
* tokens view page, view on chain, token refresh, transaction page
* transfer nfh using /account/sign (otp flow)
- transfer - signed using private key and transfer manually using postman collection
- signed with private-key-1 and verified using public-key-2 -> fails as expected


bugs:

* cache hit cache miss verbose logs - fixed
* `back` button in create account page goes back to signin page, again need to verify otp - fixed
* existing wallet 409 error in UI - fixed
* clients approval `schema` field fix (client_id -> clientId) in sandbox - fixed
* terms and condition modal db fix - `units-in` instance 

In progress: Delegation flows (including code fixes)
solana mainnet causing two entries
```


**NOTE**
`code cleanup required for scope-aproval in workflow registry`


delegations on tokens


* view on same instance worked from owner to grantee
	* labels:
	* tokens: id: value
	* `tokens: *`
	* tokens: tokenClass: NFH-T
	* tokens:tokenClassId: value -> not appending type access
	* tokens: tokenClass: * ->  not appending type access
	* tokens:metadata.symbol:USDC ->not appending type access
* grantee having view access tried to transact - failed as expected
* Cross instance user-1 (instance-1) gave view access to user-2 (instance-2): delegations flipped to active and identities appended with type access - partially right, name is invalid 
  **Token Not Found Error for user-2**
  ```json
 [
  {
    "id": "2825a3abf211b3c12c5e1e08226528e27d0f819c394e65f3106d6a017d0e89b1",
    "name": "hardik",
    "type": "owner"
  },
  {
    "id": "2825a3abf211b3c12c5e1e08226528e27d0f819c394e65f3106d6a017d0e89b1",
    "name": "hardik",
    "type": "operator"
  },
  {
    "id": "2825a3abf211b3c12c5e1e08226528e27d0f819c394e65f3106d6a017d0e89b1",
    "name": "hardik",
    "type": "creator"
  },
  {
    "id": "fc59487712bbe89b488847b77b5744fb6b815b8fc65ef2ab18149958edb61464",
    "name": "fc59487712bbe89b488847b77b5744fb6b815b8fc65ef2ab18149958edb61464",
    "type": "access"
  }
]
  ```

* same instance scenario owner gives view access to user1, now user1 gives view access to user2 ideally owner should accept this, since grantor address is taking as owner -> skipping approval
bugs
type access in not appended -> fixed 
