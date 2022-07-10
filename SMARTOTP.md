# SmartOTP

## Assumptions
Following the assumptions in SmartOTP:

1. The secret seed for OTP is only stored and kept on the Authenticator.
2. The client keeps a copy of the seed only before creating the Smart Contract wallet.
3. When the wallet is created, the client generates a large number of OTPs with seed. These OTPs are then hashed and built into a Merkle tree2. The hashed values are kept in the Client only. They will be used in computing proofs for operations. The seed is discarded after the wallet is created.
4. The Smart Contract wallet only keeps the root of the Merkle tree created in (3). The roots are sufficient to verify whether an OTP is valid.