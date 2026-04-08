---
name: ika-solana-prealpha
description: Guide for Ika dWallet on Solana pre-alpha - gRPC DWalletService, BCS request types, on-chain program (PDAs, approve_message, commit_dwallet, commit_signature), Pinocchio/Native/Anchor CPI SDKs, and @ika.xyz/pre-alpha-solana-client + @solana/kit. Use when integrating Solana ika, Solana dWallet, ika pre-alpha, dWallet Solana devnet, gRPC dWallet, approve_message Solana, MessageApproval PDA, DWalletContext, @ika.xyz/pre-alpha-solana-client, ika-dwallet-pinocchio, ika-dwallet-native, ika-dwallet-anchor, or comparing Solana pre-alpha to Sui ika-sdk.
---

# ika solana pre-alpha

Normative documentation: [solana pre-alpha docs](https://solana-pre-alpha.ika.xyz/) (mdbook sources live in [dwallet-labs/ika-pre-alpha](https://github.com/dwallet-labs/ika-pre-alpha), `docs/`). This skill summarizes workflows; load `references/` for BCS, layouts, and instruction tables.

**Docs revision:** [`references/docs-revision.md`](references/docs-revision.md) records the **git commit** this skill was aligned against for **`docs/`** only. If the mdbook sources on `main` have changed since that commit (see that file), **notify the human user** that ika-pre-alpha docs have moved ahead, this skill may be outdated, and they may wish to **disable this skill** until they obtain an updated bundle or re-verify against the live book.

## pre-alpha disclaimer (non-negotiable)

- Solana pre-alpha is for **SDK exploration and dev only**.
- **No real MPC** — a **single mock signer**, not a distributed network.
- **Do not submit real-value (mainnet / production) transactions for signing** or rely on security guarantees. Keys, trust model, and protocol are **not final**; **do not rely on key material** until mainnet.
- **Devnet resets** - state and interfaces may change without notice.
- **No warranty** - use at your own risk.

**Pass-through:** When exposing pre-alpha flows to end users, customers, or the public, do not present the stack as production MPC or stable custody. Surface the limitations above (mock signer, no security guarantees, devnet instability, non-final keys) proportionally—in copy, docs, or in-app notices. Internal-only environments where the stack is already understood need not belabor this.

## references (load on demand)

| file | when to load it |
| --- | --- |
| [`references/docs-revision.md`](references/docs-revision.md) | Tracked **`docs/`** commit vs `ika-pre-alpha` `main`; if `docs/` changed, notify user—do not patch skill files; user may disable skill until updated |
| [`references/grpc-api.md`](references/grpc-api.md) | `SubmitTransaction`, `UserSignedRequest`, BCS `DWalletRequest` / `SignedRequestData`, `ApprovalProof::Solana`, presign RPCs, enum wire values |
| [`references/account-layouts.md`](references/account-layouts.md) | PDA seeds, byte offsets, rent helper, `MessageApproval` / `DWallet`, ika system accounts |
| [`references/instructions.md`](references/instructions.md) | Instruction discriminators, account metas, ix data; voting and multisig **example** programs |
| [`references/events.md`](references/events.md) | Self-CPI event layout vs polling `MessageApproval` |
| [`references/frameworks.md`](references/frameworks.md) | `DWalletContext`, framework choice, dependencies |
| [`references/flows.md`](references/flows.md) | Ordered DKG, sign, CPI, presign, authority, verification, demo lifecycles |

## install

**TypeScript**

```bash
pnpm add @ika.xyz/pre-alpha-solana-client @solana/kit
```

**Rust (on-chain CPI)** - `ika-dwallet-*` crates from git (**source repo** row in the table below):

- Pinocchio: `ika-dwallet-pinocchio` + `pinocchio` + `pinocchio-system`
- Native: `ika-dwallet-native` + `solana-program`
- Anchor v1: `ika-dwallet-anchor` + `anchor-lang = "1"` (Anchor CLI 1.x, Solana CLI 3.x+)

**Rust (gRPC)** - `ika-grpc`, `ika-dwallet-types`, `tokio` with `rt-multi-thread`, `macros`. The sample in [`references/grpc-api.md`](references/grpc-api.md) expects `DWALLET_GRPC_URL` set to the **dWallet gRPC** value above.

**Account types** - `ika-solana-sdk-types` (published name may appear as `ika-sdk-types` in docs)

## environment (pre-alpha)

| resource | value |
| --- | --- |
| dWallet gRPC | `https://pre-alpha-dev-1.ika.ika-network.net:443` |
| Solana RPC | `https://api.devnet.solana.com` (typical) |
| dWallet program id | `87W54kGYFQ1rgWqMeu4XTPHWXWmXSQCcjm8vCTfiq1oY` |
| source repo | `https://github.com/dwallet-labs/ika-pre-alpha` |

**Canonical values:** Treat this table as the single source for program id, gRPC URL, default Solana RPC, and ika `git` remote. Other files link here; literals inside code samples are for copy-paste and must stay aligned with this table.

Devnet plus gRPC suffice for baseline integration without a local validator.

## wire quick pointers

- On-chain **curve** byte: 0 Secp256k1, 1 Secp256r1, 2 Curve25519, 3 Ristretto ([`account-layouts.md`](references/account-layouts.md), [`grpc-api.md`](references/grpc-api.md)).
- **`approve_message` signature_scheme:** 0 Ed25519, 1 Secp256k1, 2 Secp256r1 ([`instructions.md`](references/instructions.md)).
- BCS enums and mock matrix: [`grpc-api.md`](references/grpc-api.md).

## product flow (5 steps)

1. DKG over gRPC; NOA **`commit_dwallet`** ([`instructions.md`](references/instructions.md)).
2. **`transfer_ownership`** to CPI PDA or retain user authority as designed.
3. **`approve_message`** creates **MessageApproval** (CPI or direct signer).
4. Signing completes via NOA **`commit_signature`** and/or gRPC **`Signature`** response.
5. Read **MessageApproval** status and signature bytes ([`account-layouts.md`](references/account-layouts.md)).

## procedure: TypeScript off-chain (direct signer)

1. Configure `@solana/kit` RPC, subscriptions, `sendAndConfirmTransactionFactory`; program id from environment table.
2. DKG: build and sign `UserSignedRequest`, `SubmitTransaction`, handle `Attestation` ([`flows.md`](references/flows.md) flow 1, [`grpc-api.md`](references/grpc-api.md)).
3. **`approve_message`:** 67-byte layout (disc `8`, bump, keccak256(message), user pubkey, scheme); PDA `["message_approval", dwallet, message_hash]`; direct-signer metas in [`instructions.md`](references/instructions.md).
4. **`Sign`:** `ApprovalProof::Solana` from step 3 tx; fill `presign_id` and partial-signature fields per [`grpc-api.md`](references/grpc-api.md).
5. Poll **MessageApproval** (offsets 139-142) or parse events ([`events.md`](references/events.md)).
6. Presign via gRPC; query with `GetPresigns` / `GetPresignsForDWallet` ([`grpc-api.md`](references/grpc-api.md)).

**Convention:** `message_hash` in PDAs and ix data is **keccak256** of the message preimage (pre-alpha).

## procedure: on-chain CPI (pattern)

```rust
let ctx = DWalletContext {
    dwallet_program: &dwallet_program_account,
    cpi_authority: &cpi_authority_account,
    caller_program: &my_program_account,
    cpi_authority_bump: bump,
};
ctx.approve_message(
    message_approval,
    dwallet,
    payer,
    system_program,
    message_hash,
    user_pubkey,
    signature_scheme,
    message_approval_bump,
)?;
```

CPI path adds **caller_program** and **cpi_authority** ([`instructions.md`](references/instructions.md), [`frameworks.md`](references/frameworks.md)).

## instruction discriminators (dWallet program, byte 0)

Canonical table: [`instructions.md`](references/instructions.md).

Frequent path: `8` approve_message, `24` transfer_ownership, `31` commit_dwallet, `42` transfer_future_sign, `43` commit_signature. Additional: `33`-`38`, `44`-`46` (confirm in source if undocumented in book).

## PDA seeds

Definitions: [`account-layouts.md`](references/account-layouts.md) (coordinator, NEK, dwallet, message_approval, CPI authority `["__ika_cpi_authority"]` under **your** program id, optional gas deposit).

## frameworks

Selection matrix and crate setup: [`frameworks.md`](references/frameworks.md). Rust CPI crates are mutually compatible on seeds, discriminators, and layouts.

## contrast with Sui `ika-sdk`

| topic | Sui (`@ika.xyz/sdk`) | Solana pre-alpha |
| --- | --- | --- |
| mutation path | PTB + `IkaClient` | Solana transactions + gRPC `SubmitTransaction` |
| on-chain shape | Objects | PDAs + fixed layouts |
| approval link | Sui effects | `ApprovalProof::Solana { tx sig, slot }` |
| `message_hash` | chain-defined | keccak256 (pre-alpha convention) |

BCS and lifecycle detail: [`grpc-api.md`](references/grpc-api.md), [`flows.md`](references/flows.md).

## related material (optional)

If present in your skills or docs tree, use generic Solana and wallet-architecture references alongside this skill for non-ika client patterns.
