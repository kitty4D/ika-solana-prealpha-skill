# upstream: ika-pre-alpha repository

Published book: [solana pre-alpha docs](https://solana-pre-alpha.ika.xyz/) is built from the **mdbook** sources in this repo (`docs/`).

**Repository:** [github.com/dwallet-labs/ika-pre-alpha](https://github.com/dwallet-labs/ika-pre-alpha)

## what to use from the repo

| area | path (typical) | role |
| --- | --- | --- |
| mdbook docs | `docs/` | same material as the hosted book; authoritative prose and examples |
| protobuf | `proto/` | gRPC service + message shapes (BCS payloads still come from Rust types) |
| BCS / gRPC types | `crates/ika-dwallet-types/`, `crates/ika-grpc/` | Rust definitions aligned with `SubmitTransaction` |
| CPI SDKs | `chains/solana/program-sdk/pinocchio/`, `anchor/`, native if present | `ika-dwallet-pinocchio`, `ika-dwallet-anchor`, etc. |
| TS / generated clients | `chains/solana/clients/` | complements `@ika.xyz/pre-alpha-solana-client` |
| account helpers | `chains/solana/sdk/types/` | on-chain layout helpers (names may differ from npm; see repo README) |
| examples | `chains/solana/examples/` | voting, multisig, e2e patterns |
| program binary | `bin/` | prebuilt `.so` for local testing (see upstream README) |
| workspace | root `Cargo.toml`, `justfile` | build, test, doc serve commands |

## tracked revision (skill maintenance)

This skill was last aligned against **upstream `main`** at:

| field | value |
| --- | --- |
| commit (full) | `4fe19db6e95748477537241bb2e4de4832351ee4` |
| commit (short) | `4fe19db` |
| upstream commit date (UTC) | 2026-04-07 |
| recorded in skill | 2026-04-07 |

**Interpretation:** instruction layouts, program id, gRPC URL, and book text can change on `main` without notice. If `main` has moved, diff the book sources (`docs/`), `proto/`, and Solana program crates against this skill’s `references/` and [`SKILL.md`](../SKILL.md) environment table.

## how to detect drift

1. **Latest `main` SHA**

   ```bash
   git ls-remote https://github.com/dwallet-labs/ika-pre-alpha.git refs/heads/main
   ```

   Or: GitHub API `GET /repos/dwallet-labs/ika-pre-alpha/commits/main` → `sha`.

2. **Compare**

   - If `sha` ≠ tracked commit above, re-check at least: devnet **program id** and **gRPC** row in the upstream README, `docs/` print export vs this skill, and any `proto/` / program changes.

3. **Hosted book**

   The public site may lag `main`; if behavior disagrees with the skill, treat **repo `docs/` on the commit you care about** as the tie-breaker after the on-chain program.

## after upstream changes

1. Update the **tracked revision** table in this file (commit, dates).
2. Update [`../SKILL.md`](../SKILL.md) **environment** table if program id, RPC, or gRPC URL changed.
3. Refresh affected `references/*.md` sections (grpc, instructions, layouts) from repo sources.
