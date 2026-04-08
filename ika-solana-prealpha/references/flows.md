# end-to-end flows (Solana pre-alpha)

## table of contents

1. Flow 1 - DKG (create dWallet)
2. Flow 2 - direct approve_message + gRPC Sign
3. Flow 3 - CPI approve_message
4. Flow 4 - presign allocation
5. Flow 5 - authority transfer
6. Flow 6 - signature verification
7. Flow 7 - voting e2e
8. Flow 8 - multisig e2e

Normative documentation URL: [`../SKILL.md`](../SKILL.md). BCS and RPC: [`grpc-api.md`](grpc-api.md). Instruction bytes and metas: [`instructions.md`](instructions.md).

---

## flow 1 - DKG (create dWallet)

**Outcome:** dWallet public key material and on-chain **DWallet** PDA; initial **authority** per current mock (verify in `ika-pre-alpha`).

1. Read **NetworkEncryptionKey** (PDA `["network_encryption_key", noa_pubkey]`).
2. Run DKG client logic from `ika-pre-alpha` (language-specific) to produce shares and proofs.
3. Build `SignedRequestData` with `DWalletRequest::DKG` or `DKGWithPublicShare` ([`grpc-api.md`](grpc-api.md)).
4. Wrap in `UserSignedRequest` with `UserSignature` over `signed_request_data`.
5. Call gRPC `SubmitTransaction`.
6. Parse `TransactionResponseData::Attestation`.
7. Submit **`commit_dwallet`** with attestation payload; confirm **DWallet** at `["dwallet", curve_byte, public_key]`.

---

## flow 2 - direct approve_message + gRPC Sign (off-chain)

**Outcome:** `MessageApproval` filled; signature from gRPC and/or on-chain.

1. Compute `message_hash` per [`../SKILL.md`](../SKILL.md) and [`instructions.md`](instructions.md) (`approve_message`).
2. Derive MessageApproval PDA: `["message_approval", dwallet, message_hash]`.
3. Submit **approve_message** (disc 8, direct signer metas): [`instructions.md`](instructions.md).
4. Record `transaction_signature` and `slot` from the confirmed Solana transaction.
5. Allocate **presign** if required: [`grpc-api.md`](grpc-api.md) `Presign` / `PresignForDWallet`.
6. Submit `Sign` with `ApprovalProof::Solana { transaction_signature, slot }` and remaining fields per [`grpc-api.md`](grpc-api.md).
7. Handle `TransactionResponseData::Signature`.
8. Confirm completion via `MessageApproval` offsets ([`account-layouts.md`](account-layouts.md)) or **`SignatureCommitted`** ([`events.md`](events.md)).

gRPC may return a signature before or in parallel with on-chain `commit_signature`; clients should tolerate ordering variance.

---

## flow 3 - CPI approve_message (on-chain)

**Outcome:** program-controlled **authority**; **approve_message** via CPI.

1. Derive `cpi_authority = PDA(["__ika_cpi_authority"], your_program_id)`.
2. **`transfer_ownership`** (disc 24) to `cpi_authority` while still current authority.
3. Construct **`DWalletContext`** ([`frameworks.md`](frameworks.md)).
4. CPI **approve_message** with six-account layout ([`instructions.md`](instructions.md)).
5. Off-chain: gRPC `Sign` with `ApprovalProof::Solana` for that CPI transaction.
6. Observe completion as in flow 2 step 8.

**Testing:** Mollusk does not exercise cross-program CPI to the dWallet program; use LiteSVM or devnet e2e for CPI coverage.

---

## flow 4 - presign allocation

Request and response shapes: [`grpc-api.md`](grpc-api.md) (`Presign`, `PresignForDWallet`, `GetPresigns`, `GetPresignsForDWallet`). Persist `presign_id` for `Sign`.

---

## flow 5 - authority transfer

- **User → program:** `transfer_ownership` signer path; `new_authority` = CPI PDA bytes.
- **On-chain handoff:** CPI `transfer_ownership` when logic allows.

**AuthorityTransferred** payload: [`events.md`](events.md).

---

## flow 6 - signature verification (off-chain)

**Ed25519 (Curve25519 dWallet)**

```rust
use ed25519_dalek::{Signature, VerifyingKey};

let verifying_key = VerifyingKey::from_bytes(&dwallet_public_key)?;
let signature = Signature::from_bytes(&signature_bytes.try_into().unwrap())?;
verifying_key.verify_strict(message_as_signed, &signature)?;
```

Use the preimage your application defines as signed; the pre-alpha book may show `message_hash` as the verified buffer for examples only.

**Secp256k1**

```rust
use secp256k1::{Message, PublicKey, Secp256k1, ecdsa::Signature};

let secp = Secp256k1::verification_only();
let pubkey = PublicKey::from_slice(&dwallet_public_key)?;
let message = Message::from_digest(message_hash);
let signature = Signature::from_compact(&signature_bytes)?;
secp.verify_ecdsa(&message, &signature, &pubkey)?;
```

---

## flow 7 - voting e2e

1. DKG + **commit_dwallet**
2. **transfer_ownership** to voting program CPI PDA
3. **create_proposal**
4. **cast_vote** until quorum; final vote supplies CPI accounts for **approve_message**
5. **MessageApproval** pending
6. **commit_signature**
7. Read signature at MessageApproval offset 142 (length at 140)

---

## flow 8 - multisig e2e

1. DKG  
2. **transfer_ownership** to multisig program CPI PDA  
3. **CreateMultisig**  
4. **CreateTransaction**  
5. **Approve** until threshold  
6. **MessageApproval** + gRPC presign + gRPC sign  
7. Optional: **Reject** until `rejection_threshold = member_count - threshold + 1` (see multisig docs in book)
