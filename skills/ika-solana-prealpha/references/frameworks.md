# frameworks - TypeScript + Rust CPI SDKs

## table of contents

1. TypeScript (`@ika.xyz/pre-alpha-solana-client` + `@solana/kit`)
2. Pinocchio (`ika-dwallet-pinocchio`)
3. Native (`ika-dwallet-native`)
4. Anchor v1 (`ika-dwallet-anchor`)
5. Comparison table, interoperability

Pinocchio, Native, and Anchor CPI crates share **instruction encodings**, **account order**, and **`CPI_AUTHORITY_SEED`**.

**Git remote for `ika-dwallet-*`:** [`../SKILL.md`](../SKILL.md) **source repo** row. The Pinocchio snippet below shows one full `[dependencies]` example; Native and Anchor add the same `git` URL on their crate (keep it identical to that table).

---

## TypeScript - `@ika.xyz/pre-alpha-solana-client` + `@solana/kit`

Install: [`../SKILL.md`](../SKILL.md). Typical Solana Kit entrypoints: `createSolanaRpc`, `pipe`, `createTransactionMessage`, `signTransactionMessageWithSigners`, `sendAndConfirmTransactionFactory`, `getProgramDerivedAddress`, `getAddressEncoder`, `getUtf8Encoder`.

### approve_message (direct authority)

Derive **DWallet** PDA with chunked seeds per [`account-layouts.md`](account-layouts.md) (not a single `[curve, pk]` pair). Derive MessageApproval PDA with seeds `message_approval`, encoded dwallet address, `message_hash` (32 bytes). Instruction data: 67 bytes per [`instructions.md`](instructions.md).

```typescript
import { address, getAddressEncoder, getProgramDerivedAddress, getUtf8Encoder } from "@solana/kit";

// Program id: paste from ../SKILL.md environment table (must stay in sync).
const DWALLET_PROGRAM = address("87W54kGYFQ1rgWqMeu4XTPHWXWmXSQCcjm8vCTfiq1oY");
const utf8 = getUtf8Encoder();
const addressEncoder = getAddressEncoder();
const [messageApprovalPda, messageApprovalBump] = await getProgramDerivedAddress({
  seeds: [utf8.encode("message_approval"), addressEncoder.encode(dwalletAddress), messageHash],
  programAddress: DWALLET_PROGRAM,
});

const data = new Uint8Array(67);
data[0] = 8;
data[1] = messageApprovalBump;
data.set(messageHash, 2);
data.set(userPubkey, 34);
data[66] = 0; // Ed25519
```

### transfer_ownership (disc 24)

```typescript
const tdata = new Uint8Array(33);
tdata[0] = 24;
tdata.set(newAuthorityBytes, 1);
// metas: current_authority signer, dwallet writable
```

### gRPC from TypeScript

`ika_dwallet.proto` in the repo; endpoint in `SKILL.md`. Serialize `UserSignedRequest` per [`grpc-api.md`](grpc-api.md).

---

## Pinocchio - `ika-dwallet-pinocchio`

Use for `#![no_std]`, minimal binary size, CU-sensitive programs.

**deps**

```toml
ika-dwallet-pinocchio = { git = "https://github.com/dwallet-labs/ika-pre-alpha" }
pinocchio = "0.10"
pinocchio-system = "0.5"
```

**DWalletContext**

```rust
use ika_dwallet_pinocchio::DWalletContext;

let ctx = DWalletContext {
    dwallet_program: &dwallet_program_account,
    cpi_authority: &cpi_authority_account,
    caller_program: &my_program_account,
    cpi_authority_bump: bump,
};
```

**CPI authority**

```rust
use ika_dwallet_pinocchio::CPI_AUTHORITY_SEED;
let (cpi_authority, bump) = pinocchio::Address::find_program_address(
    &[CPI_AUTHORITY_SEED],
    program_id,
);
```

**Methods:** `approve_message`, `transfer_dwallet`, `transfer_future_sign` (signatures in crate docs).

**Types:** `AccountView`, `pinocchio::entrypoint!`, `pinocchio::cpi::invoke_signed`.

---

## Native - `ika-dwallet-native`

Use for existing `solana-program` codebases (`std`, `AccountInfo`).

**deps:** add `ika-dwallet-native = { git = ... }` using the **source repo** URL from [`../SKILL.md`](../SKILL.md), plus:

```toml
solana-program = "2.2"
solana-system-interface = "1"
```

**DWalletContext** uses `&AccountInfo<'info>`; same three CPI methods as Pinocchio.

**CPI authority**

```rust
use ika_dwallet_native::CPI_AUTHORITY_SEED;
let (cpi_authority, bump) = Pubkey::find_program_address(&[CPI_AUTHORITY_SEED], &program_id);
```

| topic | Pinocchio | Native |
| --- | --- | --- |
| account type | `AccountView` | `AccountInfo` |
| std | no_std | std |
| rent | pinocchio helpers | `Rent::get()?.minimum_balance` |
| CPI | pinocchio invoke | `solana_program::program::invoke_signed` |
| iteration | fixed indices | `next_account_info` |

---

## Anchor v1 - `ika-dwallet-anchor`

Use for Anchor 1.x programs and declarative account validation.

**deps:** add `ika-dwallet-anchor = { git = ... }` using the **source repo** URL from [`../SKILL.md`](../SKILL.md), plus:

```toml
anchor-lang = "1"
```

**Tooling:** Anchor CLI 1.x, Solana CLI 3.x (see upstream book for `avm` pins).

**DWalletContext**

```rust
use ika_dwallet_anchor::{DWalletContext, CPI_AUTHORITY_SEED};

let ctx = DWalletContext {
    dwallet_program: dwallet_program.to_account_info(),
    cpi_authority: cpi_authority.to_account_info(),
    caller_program: program.to_account_info(),
    cpi_authority_bump: bump,
};
```

**Methods**

```rust
ctx.approve_message(
    &message_approval.to_account_info(),
    &dwallet.to_account_info(),
    &payer.to_account_info(),
    &system_program.to_account_info(),
    message_hash,
    user_pubkey,
    signature_scheme,
    bump,
)?;
ctx.transfer_dwallet(&dwallet.to_account_info(), &new_authority)?;
ctx.transfer_future_sign(&partial_user_sig.to_account_info(), &new_authority)?;
```

Anchor v1 patterns from book: `UncheckedAccount`, `InitSpace`, consolidated `#[error_code]`.

---

## comparison table

| | Pinocchio | Native | Anchor | TypeScript |
| --- | --- | --- | --- | --- |
| language | Rust | Rust | Rust | TS |
| runs | on-chain | on-chain | on-chain | off-chain |
| account type | `AccountView` | `AccountInfo` | `Account` / `UncheckedAccount` | addresses + bytes |
| std | no | yes | yes | n/a |
| best for | max CU / size | legacy svm | rapid iteration | clients, automation |

---

## interoperability

Same `CPI_AUTHORITY_SEED`, discriminators, and per-instruction account order across Rust CPI crates. Example programs target the dWallet program id from [`../SKILL.md`](../SKILL.md) and the same PDA derivations on devnet.
