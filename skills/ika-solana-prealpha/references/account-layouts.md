# account layouts (pre-alpha)

## table of contents

1. Rent helper (book / examples)
2. dWallet program accounts (coordinator, NEK, **DWallet chunked PDA seeds**, MessageApproval)
3. CPI authority PDA
4. Ika system accounts (`ika-solana-sdk-types`)
5. Voting + multisig example accounts
6. Summary table
7. TypeScript read hints

Account layouts follow the dWallet program and `ika-solana-sdk-types` documentation.

**Prefix:** typed accounts use **`discriminator` (1 byte) + `version` (1 byte)** then payload.

**dWallet program id:** see environment table in [`../SKILL.md`](../SKILL.md).

---

## rent helper (book / examples)

Example approximation from the book:

```rust
fn minimum_balance(data_len: usize) -> u64 {
    (data_len as u64 + 128) * 6960
}
```

On-chain: use `Rent::get()?.minimum_balance(data_len)`. Clients: use RPC rent where appropriate. Sizes below support estimation.

---

## dWallet program accounts

### type discriminators

| disc (type) | account |
| --- | --- |
| 1 | DWalletCoordinator |
| 3 | NetworkEncryptionKey |
| 14 | MessageApproval |

`DWallet` uses the on-chain type discriminator at byte 0; PDA seeds are **`["dwallet", ...chunks]`** where `chunks` splits **`(curve_byte || raw_public_key)`** into **32-byte** pieces (Solana `MAX_SEED_LEN`), each passed as its own seed. Concatenate curve (1 byte) and pubkey, then `chunks(32)`.

| pubkey | payload `curve \|\| pk` | chunk sizes |
| --- | --- | --- |
| 32 bytes (Ed25519 / Curve25519 / Ristretto) | 33 bytes | `[32, 1]` |
| 33 bytes (compressed Secp256k1 / Secp256r1) | 34 bytes | `[32, 2]` |
| 65 bytes (uncompressed SEC1) | 66 bytes | `[32, 32, 2]` |

---

### DWalletCoordinator - 116 bytes

- **PDA seeds:** `["dwallet_coordinator"]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`1`) |
| 1 | version | 1 (`1`) |
| 2 | coordinator fields | 114 |

---

### NetworkEncryptionKey - 164 bytes

- **PDA seeds:** `["network_encryption_key", noa_pubkey]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`3`) |
| 1 | version | 1 (`1`) |
| 2 | NEK fields | 162 |

Read NEK bytes from chain for `dwallet_network_encryption_public_key` in DKG requests.

---

### DWallet

- **PDA seeds:** `["dwallet", chunks_of(curve_byte || public_key)]` (32-byte chunks; see table above)

| offset | field | size | notes |
| --- | --- | --- | --- |
| 0 | discriminator | 1 | type |
| 1 | version | 1 | `1` |
| 2 | authority | 32 | user or CPI PDA |
| 34 | public_key | 65 | padded |
| 99 | public_key_len | 1 | 32 or 33 typical |
| 100 | curve | 1 | 0 Secp256k1, 1 Secp256r1, 2 Curve25519, 3 Ristretto |
| 101 | is_imported | 1 | DKG vs imported |

---

### MessageApproval - 287 bytes (2 + 285)

- **PDA seeds:** `["message_approval", dwallet_pubkey, message_hash]`
- **`message_hash`:** must match `approve_message` ix data in [`instructions.md`](instructions.md); preimage rule in [`../SKILL.md`](../SKILL.md).

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`14`) |
| 1 | version | 1 (`1`) |
| 2 | dwallet | 32 |
| 34 | message_hash | 32 |
| 66 | user_pubkey | 32 |
| 98 | signature_scheme | 1 |
| 99 | caller_program | 32 |
| 131 | cpi_authority | 32 |
| 139 | status | 1 |
| 140 | signature_len | 2 (LE u16) |
| 142 | signature | 128 (padded) |

**status:** `0` pending, `1` signed.

**Signature bytes:** `len = u16::from_le_bytes(data[140..142].try_into()?)`, then `data[142..142+len]`.

---

## CPI authority PDA

- **seeds:** `["__ika_cpi_authority"]`
- **program id:** calling program (not the dWallet program)
- Must equal `dwallet.authority` for CPI `approve_message` / `transfer_ownership` paths.

---

## Ika system accounts (`ika-solana-sdk-types`)

### SystemState - 365 bytes

- **seeds:** `["ika_system_state"]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`1`) |
| 1 | version | 1 |
| 2 | epoch | 8 (LE u64) |
| 10 | … | … |
| 34 | authority | 32 |

### Validator - 973 bytes

- **seeds:** `["validator", identity_pubkey]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`2`) |
| 1 | version | 1 |
| 2 | identity | 32 |
| 98 | state | 1 |
| 159 | ika_balance | 8 (LE u64) |

### StakeAccount - 115 bytes

- **seeds:** `["stake_account", stake_id_le_bytes]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`3`) |
| 1 | version | 1 |
| 2 | owner | 32 |
| 74 | principal | 8 |
| 98 | state | 1 |

### ValidatorList

- **seeds:** `["validator_list"]`

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`4`) |
| 1 | version | 1 |
| 2 | validator_count | 4 (LE u32) |
| 6 | active_count | 4 (LE u32) |

---

## voting example accounts (separate program)

### Proposal - 195 bytes

- **seeds:** `["proposal", proposal_id]`

Fields include: `proposal_id`, `dwallet`, `message_hash`, `user_pubkey`, `signature_scheme`, vote counts, `quorum`, `status`, `msg_approval_bump`, `bump`.

### VoteRecord - 69 bytes

- **seeds:** `["vote", proposal_id, voter]`

---

## multisig example accounts (separate program)

| account | seeds | size |
| --- | --- | --- |
| Multisig | `["multisig", create_key]` | 395 |
| Transaction | `["transaction", multisig, tx_index_le]` | 432 |
| ApprovalRecord | `["approval", transaction, member]` | 68 |

Field-level layouts for multisig accounts: [`instructions.md`](instructions.md) multisig section.

---

## summary table

| account | disc | size | seeds | program |
| --- | --- | --- | --- | --- |
| DWalletCoordinator | 1 | 116 | `["dwallet_coordinator"]` | dWallet |
| NetworkEncryptionKey | 3 | 164 | `["network_encryption_key", noa]` | dWallet |
| MessageApproval | 14 | 287 | `["message_approval", dwallet, hash]` | dWallet |
| DWallet | (type) | see above | `["dwallet", chunks_of(curve \|\| pk)]` | dWallet |
| SystemState | 1 | 365 | `["ika_system_state"]` | Ika system |
| Validator | 2 | 973 | `["validator", identity]` | Ika system |
| StakeAccount | 3 | 115 | `["stake_account", stake_id]` | Ika system |
| ValidatorList | 4 | 18+ | `["validator_list"]` | Ika system |

---

## TypeScript read hints (`@solana/kit` + rpc)

Decode `getAccountInfo` data (e.g. base64), then:

- **MessageApproval:** status `data[139]`; `signature_len` = LE `u16` at 140; signature at 142 for `signature_len` bytes.
- **dWallet:** use the offset table in this file (from byte 2). Validate against current devnet `getAccountInfo` if documentation and chain diverge.
