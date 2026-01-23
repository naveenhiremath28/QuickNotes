

*Notes / Required Fixes

- Mint operation:
  In the token_transactions table → participants field, "did" instead of "address" during insertion.

- Transfer operation:
  In the token_transactions table → participants field, the sender address is incorrectly replaced with the receiver’s address during insertion.

- Postman collection updates:
  - Need to remove the "memo" field from token/transfer (INRT, RWA-NFT, ART-NFT).
  - Need to Add an "amount" field to token/transfer (RWA-NFT).



services/token.go -> 338 -> its should be owner not the initiator
