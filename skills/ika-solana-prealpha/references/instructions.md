# instruction reference (dWallet program + examples)

## table of contents

1. dWallet program (groups, full discriminator table)
2. approve_message, transfer_ownership, commit_dwallet, transfer_future_sign, commit_signature
3. Voting example program (create_proposal, cast_vote)
4. Multisig example program

Byte **0** of dWallet program instruction data is the **discriminator**.

**Program id:** [`../SKILL.md`](../SKILL.md) environment table (dWallet program id).

---

## dWallet program - groups

| group | discriminators | instructions |
| --- | --- | --- |
| Message | 8 | approve_message |
| Ownership | 24 | transfer_ownership |
| DKG | 31 | commit_dwallet |
| Extended | 33-38 | future sign, share commits, gas helpers |
| Signing | 42-43 | transfer_future_sign, commit_signature |
| Withdraw / init | 44-46 | RequestWithdraw, Withdraw, Initialize |

### full discriminator table

| disc | name |
| --- | --- |
| 8 | approve_message |
| 24 | transfer_ownership |
| 31 | commit_dwallet |
| 33 | CommitFutureSign |
| 34 | CommitEncryptedUserSecretKeyShare |
| 35 | CommitPublicUserSecretKeyShare |
| 36 | CreateDeposit |
| 37 | TopUp |
| 38 | SettleGas |
| 42 | transfer_future_sign |
| 43 | commit_signature |
| 44 | RequestWithdraw |
| 45 | Withdraw |
| 46 | Initialize |

Instruction data for discriminators **33-38** and **44-46** is not fully documented in the public book; read the **source repo** in [`../SKILL.md`](../SKILL.md) for layouts.

---

## approve_message (disc 8)

Creates **MessageApproval** PDA. **`message_hash`:** **opaque 32-byte uniqueness key** for the PDA — must be **`keccak256(preimage)`** regardless of destination chain. Independent of gRPC **`Sign`** `hash_scheme` (which controls the digest actually signed). Same rule as [`../SKILL.md`](../SKILL.md).

### data - 67 bytes

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`8`) |
| 1 | bump | 1 |
| 2 | message_hash | 32 |
| 34 | user_pubkey | 32 |
| 66 | signature_scheme | 1 (0 Ed25519, 1 Secp256k1, 2 Secp256r1) |

### accounts - CPI path (6 metas)

| # | account | writable | signer | notes |
| --- | --- | --- | --- | --- |
| 0 | message_approval | yes | no | empty PDA |
| 1 | dwallet | no | no | |
| 2 | caller_program | no | no | executable |
| 3 | cpi_authority | no | yes | PDA `["__ika_cpi_authority"]` for caller |
| 4 | payer | yes | yes | rent |
| 5 | system_program | no | no | |

**Checks:** `cpi_authority == PDA(["__ika_cpi_authority"], caller_program)` and `dwallet.authority == cpi_authority`.

### accounts - direct signer path (5 metas)

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | message_approval | yes | no |
| 1 | dwallet | no | no |
| 2 | authority | no | yes |
| 3 | payer | yes | yes |
| 4 | system_program | no | no |

---

## transfer_ownership (disc 24)

### data - 33 bytes

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`24`) |
| 1 | new_authority | 32 |

### accounts - signer path

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | current_authority | no | yes |
| 1 | dwallet | yes | no |

### accounts - CPI path

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | caller_program | no | no |
| 1 | cpi_authority | no | yes |
| 2 | dwallet | yes | no |

---

## commit_dwallet (disc 31) - NOA only

Creates **DWallet** PDA after DKG attestation.

### accounts

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | coordinator | no | no |
| 1 | nek | no | no |
| 2 | noa | no | yes |
| 3 | dwallet | yes | no |
| 4 | authority | no | no |
| 5 | payer | yes | yes |
| 6 | system_program | no | no |

### data (from book)

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 |
| 1 | curve | 1 |
| 2 | is_imported | 1 |
| 3 | public_key_len | 1 |
| 4 | public_key | 65 |
| 69 | bump | 1 |
| 70 | public_output_len | 2 (LE u16) |
| 72 | public_output | 256 |
| 328 | noa_signature | 64 |

---

## transfer_future_sign (disc 42) - CPI path

### accounts

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | partial_user_sig | yes | no |
| 1 | caller_program | no | no |
| 2 | cpi_authority | no | yes |

### data - 33 bytes

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 (`42`) |
| 1 | new_completion_authority | 32 |

---

## commit_signature (disc 43) - NOA only

Writes signature into **MessageApproval**.

### accounts

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | message_approval | yes | no |
| 1 | nek | no | no |
| 2 | noa | no | yes |

### data

| offset | field | size |
| --- | --- | --- |
| 0 | discriminator | 1 |
| 1 | signature_len | 2 (LE u16) |
| 3 | signature | 128 |

---

# voting example program (not dWallet program)

## create_proposal (disc 0)

### accounts

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | proposal | yes | no |
| 1 | dwallet | no | no |
| 2 | creator | no | yes |
| 3 | payer | yes | yes |
| 4 | system_program | no | no |

### data - 103 bytes

`proposal_id(32) | message_hash(32) | user_pubkey(32) | signature_scheme(1) | quorum(4 LE u32) | message_approval_bump(1) | bump(1)`

---

## cast_vote (disc 1)

### accounts - base

| # | account | writable | signer |
| --- | --- | --- | --- |
| 0 | proposal | yes | no |
| 1 | vote_record | yes | no |
| 2 | voter | no | yes |
| 3 | payer | yes | yes |
| 4 | system_program | no | no |

### accounts - when quorum reached (+5)

| # | account |
| --- | --- |
| 5 | message_approval |
| 6 | dwallet |
| 7 | caller_program (voting program) |
| 8 | cpi_authority |
| 9 | dwallet_program |

### data - 35 bytes

`proposal_id(32) | vote(1) | vote_record_bump(1) | cpi_authority_bump(1)`

---

# multisig example program

| disc | instruction |
| --- | --- |
| 0 | CreateMultisig |
| 1 | CreateTransaction |
| 2 | Approve |
| 3 | Reject |

**Multisig** PDA `["multisig", create_key]`, 395 bytes: disc, version, create_key, threshold (u16), member_count (u16), tx_index (u32), dwallet, bump, members (10×32).

**Transaction** PDA `["transaction", multisig, tx_index_le]`, 432 bytes: `message_hash` offset 70; `approval_count` / `rejection_count` from offset 135; `message_data_len` at 174; `message_data` at 176 (256 bytes).

**ApprovalRecord** `["approval", transaction, member]`, 68 bytes.

At `approval_count >= threshold`, the reference program CPI-calls **approve_message** and may call **transfer_future_sign**.
