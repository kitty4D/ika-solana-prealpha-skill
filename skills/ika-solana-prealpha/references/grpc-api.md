# gRPC API (pre-alpha)

## table of contents

1. Service definition
2. UserSignedRequest, UserSignature, SignedRequestData
3. DWalletRequest (BCS enum), DKG / Sign fields, mock matrix
4. ApprovalProof
5. TransactionResponse, TransactionResponseData
6. Query RPCs (GetPresigns, GetPresignsForDWallet)
7. BCS notes, minimal Rust client
8. Crypto parameter enums (BCS)

Mutations use **`SubmitTransaction`**. Protobuf carries **BCS** payloads in `bytes` fields.

**Endpoints and Rust crate git remote:** [`../SKILL.md`](../SKILL.md) environment + install sections (dWallet gRPC URL, **source repo**, **Rust (gRPC)** crate names).

**TypeScript:** `@grpc/grpc-js`, Connect-RPC, or grpc-web; serialize with `@mysten/bcs` or another layout-compatible BCS implementation aligned with `ika-dwallet-types`.

---

## service definition

```protobuf
service DWalletService {
  rpc SubmitTransaction(UserSignedRequest) returns (TransactionResponse);
  rpc GetPresigns(GetPresignsRequest) returns (GetPresignsResponse);
  rpc GetPresignsForDWallet(GetPresignsForDWalletRequest) returns (GetPresignsResponse);
}
```

---

## UserSignedRequest

```protobuf
message UserSignedRequest {
  bytes user_signature = 1;      // BCS UserSignature
  bytes signed_request_data = 2; // BCS SignedRequestData
}
```

| field | description |
| --- | --- |
| `user_signature` | BCS `UserSignature` (scheme + signature + public key) |
| `signed_request_data` | BCS `SignedRequestData` (signed payload) |

Validators check that `user_signature` covers `signed_request_data`.

---

## UserSignature (BCS enum)

```rust
pub enum UserSignature {
    Ed25519 {
        signature: Vec<u8>,   // 64
        public_key: Vec<u8>,  // 32
    },
    Secp256k1 {
        signature: Vec<u8>,   // 64
        public_key: Vec<u8>,  // 33 compressed
    },
    Secp256r1 {
        signature: Vec<u8>,   // 64
        public_key: Vec<u8>,  // 33 compressed
    },
}
```

---

## SignedRequestData

```rust
pub struct SignedRequestData {
    pub session_identifier_preimage: [u8; 32],
    pub epoch: u64,
    pub chain_id: ChainId,
    pub intended_chain_sender: Vec<u8>,
    pub request: DWalletRequest,
}
```

| field | role |
| --- | --- |
| `session_identifier_preimage` | 32-byte nonce (replay control) |
| `epoch` | ika epoch |
| `chain_id` | `Solana` or `Sui` |
| `intended_chain_sender` | caller address on target chain (bytes) |
| `request` | operation payload |

---

## DWalletRequest (BCS enum)

Exact struct fields: upstream book and `ika-dwallet-types`.

| variant | pre-alpha mock |
| --- | --- |
| `DKG { ... }` | supported; on-chain commit + authority per current mock behavior |
| `DKGWithPublicShare { ... }` | supported (mock equivalent to DKG) |
| `Sign { ... }` | supported |
| `ImportedKeySign { ... }` | supported (mock same as Sign) |
| `Presign { ... }` | supported |
| `PresignForDWallet { ... }` | supported |
| `ImportedKeyVerification` | error |
| `ReEncryptShare` | error |
| `MakeSharePublic` | error |
| `FutureSign` | error |
| `SignWithPartialUserSig` | error |
| `ImportedKeySignWithPartialUserSig` | error |

### DKG fields (representative)

- `dwallet_network_encryption_public_key`
- `curve: DWalletCurve`
- `centralized_public_key_share_and_proof`
- `encrypted_centralized_secret_share_and_proof`
- `encryption_key`
- `user_public_output`
- `signer_public_key`

`DKGWithPublicShare` uses `public_user_secret_key_share` instead of the encrypted share fields.

### Sign fields

```rust
DWalletRequest::Sign {
    message: Vec<u8>,
    curve: DWalletCurve,
    signature_algorithm: DWalletSignatureAlgorithm,
    hash_scheme: DWalletHashScheme,
    presign_id: Vec<u8>,
    message_centralized_signature: Vec<u8>,
    approval_proof: ApprovalProof,
}
```

Pre-alpha mock: `signature_algorithm` and `hash_scheme` are **accepted and ignored**.

---

## ApprovalProof

```rust
pub enum ApprovalProof {
    Solana {
        transaction_signature: Vec<u8>,
        slot: u64,
    },
    Sui {
        effects_certificate: Vec<u8>,
    },
}
```

After Solana `approve_message`, set **`Solana`** to the signature bytes and **slot** of the transaction that created the `MessageApproval`.

---

## TransactionResponse

```protobuf
message TransactionResponse {
  bytes response_data = 1; // BCS TransactionResponseData
}
```

## TransactionResponseData (BCS)

```rust
pub enum TransactionResponseData {
    Signature { signature: Vec<u8> },
    Attestation {
        attestation_data: Vec<u8>,
        network_signature: Vec<u8>,
        network_pubkey: Vec<u8>,
        epoch: u64,
    },
    Presign {
        presign_id: Vec<u8>,
        presign_data: Vec<u8>,
        epoch: u64,
    },
    Error { message: String },
}
```

| variant | use |
| --- | --- |
| `Signature` | `Sign` / `ImportedKeySign` (64-byte algorithms as documented) |
| `Attestation` | `DKG` / `DKGWithPublicShare` → input to `commit_dwallet` |
| `Presign` | `Presign` / `PresignForDWallet` |
| `Error` | handle before other variants |

---

## query RPCs

### GetPresignsRequest

```protobuf
message GetPresignsRequest {
  bytes user_pubkey = 1;
}
```

### GetPresignsForDWalletRequest

```protobuf
message GetPresignsForDWalletRequest {
  bytes user_pubkey = 1;
  bytes dwallet_id = 2;
}
```

### GetPresignsResponse

```protobuf
message GetPresignsResponse {
  repeated PresignInfo presigns = 1;
}

message PresignInfo {
  bytes presign_id = 1;
  bytes dwallet_id = 2;
  uint32 curve = 3;
  uint32 signature_scheme = 4;
  uint64 epoch = 5;
}
```

---

## BCS notes

Match `ika-dwallet-types` tag ordering; avoid hand-rolled enums when generated bindings exist.

- Rust: `bcs::to_bytes` / `bcs::from_bytes`
- TypeScript: generated code or `@mysten/bcs` with verified layouts

---

## minimal Rust client shape

```rust
use ika_grpc::d_wallet_service_client::DWalletServiceClient;
use ika_grpc::UserSignedRequest;
use std::env;

// DWALLET_GRPC_URL = "dWallet gRPC" value from ../SKILL.md environment table.
let mut client = DWalletServiceClient::connect(env::var("DWALLET_GRPC_URL").unwrap()).await?;

let resp = client.submit_transaction(UserSignedRequest {
    user_signature: bcs::to_bytes(&user_sig)?,
    signed_request_data: bcs::to_bytes(&signed_data)?,
}).await?;

let tx_response = resp.into_inner();
let result: TransactionResponseData = bcs::from_bytes(&tx_response.response_data)?;
```

---

## crypto parameter enums (BCS)

**DWalletCurve:** Secp256k1, Secp256r1, Curve25519, Ristretto

**DWalletSignatureAlgorithm (index):** ECDSASecp256k1(0), ECDSASecp256r1(1), Taproot(2), EdDSA(3), SchnorrkelSubstrate(4)

**DWalletHashScheme (index):** Keccak256(0), SHA256(1), DoubleSHA256(2), SHA512(3), Merlin(4)

**ChainId:** Solana, Sui
