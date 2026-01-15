
>2025.12.23.GA tag


Approach

to make changes in keycloak token we need to have keycloak encryption key(we get it from keycloak database) so, when we have that key, we can modify the token claims and again we can sign for that token








we need to create token/get api, similarly how we have for accounts
have proper routes, controller, services, schema validations etc
as of now only implement token/get


request :
```json
{
  "context": {
    "id": "api.token.get",
    "version": "1.0",
    "ts": "2025-01-04T10:00:00Z",
    "msgId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "developerToken": "dev_pk_..."
  },
  "payload": {
    "tokenId": "urn:uuid:a1b2c3d4-e5f6-7890-abcd-1234567890ab"
  }
}
```

response:

```json
{
  "context": {
    "id": "api.token.get",
    "version": "1.0",
    "ts": "2026-01-14T11:12:08.484Z",
    "msgId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "status": "successful",
    "error": {
      "code": "RESOURCE_NOT_FOUND",
      "message": "string"
    }
  },
  "response": {
    "id": "string",
    "metadata": {
      "tokenStandard": "ERC-3643",
      "name": "USD Coin",
      "symbol": "USDC",
      "decimals": 6,
      "fungibility": "fungible",
      "tokenClass": "USDC",
      "version": "1.0.0",
      "status": "active",
      "valuationType": "fixed",
      "stateModel": "native",
      "flags": {
        "transactable": true,
        "transferable": true,
        "locked": false,
        "revocable": false
      },
      "tags": [
        "string"
      ],
      "description": "string",
      "externalUrls": [
        "string"
      ],
      "createdAt": "2026-01-14T11:12:08.485Z",
      "updatedAt": "2026-01-14T11:12:08.485Z"
    },
    "data": {
      "additionalProp1": {}
    },
    "claims": [
      {
        "id": "string",
        "@context": "string",
        "type": "string",
        "issuer": "string",
        "issuanceDate": "2026-01-14T11:12:08.485Z",
        "expirationDate": "2026-01-14T11:12:08.485Z",
        "visibility": "public",
        "credentialSubject": {
          "id": "string",
          "additionalProp1": {}
        },
        "proof": {
          "type": "string",
          "created": "2026-01-14T11:12:08.485Z",
          "verificationMethod": "string",
          "proofPurpose": "string",
          "jws": "string"
        },
        "status": "asserted"
      }
    ],
    "identities": [
      {
        "id": "string",
        "type": "issuer",
        "roles": [
          "string"
        ]
      }
    ],
    "state": {
      "status": "active",
      "effectiveFrom": "2026-01-14T11:12:08.485Z",
      "effectiveUntil": "2026-01-14T11:12:08.485Z",
      "supply": {
        "totalSupply": "string",
        "circulatingSupply": "string"
      },
      "lockState": {
        "locked": true,
        "lockReason": "string",
        "lockedBy": "string",
        "lockedAt": "2026-01-14T11:12:08.485Z"
      },
      "restrictions": {
        "jurisdiction": [
          "string"
        ],
        "allowedUseCases": [
          "string"
        ]
      },
      "customState": {
        "additionalProp1": {}
      },
      "stateCommitment": "string"
    }
  }
}
```



for schema validation: https://github.com/finternet-io/specs/blob/alpha/api/token-interfaces.yaml






metada and data can refer from table_classes schema field



https://www.youtube.com/watch?v=7JRVZJDyyxw