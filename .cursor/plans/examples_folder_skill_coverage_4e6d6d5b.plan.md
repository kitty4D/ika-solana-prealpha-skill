---
name: ika-solana-prealpha-examples-and-audit
overview: "Single rollout: (A) `references/examples.md` + cross-links; (B) audit via skill subcommands `/ika-solana-prealpha audit` and `/ika-solana-prealpha audit-force` (SKILL.md branches like i-impeccable craft/teach/extract), plus Node `scripts/audit-ika-solana-prealpha.mjs` — no separate `.cursor/commands` file unless Cursor does not expose `/ika-solana-prealpha` for this skill."
todos:
  - id: add-examples-md
    content: "Add `references/examples.md`: repo link, table for voting/multisig/protocols-e2e/_shared, framework subdirs (note multisig-only `react`, no `voting/react`), 2–3 lines on `e2e-protocols` vs on-chain examples."
    status: pending
  - id: wire-crossrefs
    content: Update `SKILL.md` (refs table), `flows.md` (flows 7–8), `frameworks.md` (one bullet), `instructions.md` (voting/multisig verify pointers) to link `examples.md`.
    status: pending
  - id: optional-description-cso
    content: "If under 1024 chars total frontmatter: extend YAML `description` with `chains/solana/examples`, `protocols-e2e`, `e2e-protocols` triggers only (no workflow)."
    status: pending
  - id: verify-writing-skills
    content: "Run retrieval check: prompt answers must locate multisig+quasar and protocols-e2e paths; grep skill for bogus `voting/react`; optional `wc -w` on hub + new file to stay lean."
    status: pending
  - id: skill-audit-modes
    content: "In `SKILL.md`, add **Audit mode** and **Audit-force mode** sections (same pattern as `i-impeccable` craft/teach/extract): `/ika-solana-prealpha audit` → docs-revision gate then script + checklist; `audit-force` → skip gate with explicit stale warning, then same script + checklist. Optional one-line in YAML `description` for discoverability (CSO tradeoff vs writing-skills ‘no workflow in description’ — keep minimal)."
    status: pending
  - id: node-script
    content: "Add `scripts/audit-ika-solana-prealpha.mjs` (Node stdlib): drift check + deps + canonical strings; support `--force` (or env) matching `audit-force` so gate is one source of truth."
    status: pending
  - id: skill-hub-audit-pointer
    content: "Hub: one line pointing to `/ika-solana-prealpha audit` / `audit-force` + `scripts/…mjs` (no separate commands folder unless fallback needed)."
    status: pending
isProject: true
---

# ika-solana-prealpha: examples coverage + skill audit modes

**Open this file from the workspace** (`.cursor/plans/…`) if `%USERPROFILE%\.cursor\plans\…` fails in the editor.

Merged from **examples folder skill coverage** and **audit-ika-solana-prealpha command**. Execute as one session / one PR if you like.

---

## Part A — Cover `chains/solana/examples` in the skill

### What upstream contains (verified on `main`)

| Path | Role |
| --- | --- |
| [`chains/solana/examples/_shared`](https://github.com/dwallet-labs/ika-pre-alpha/tree/main/chains/solana/examples/_shared) | Shared TS / setup helpers for e2e clients |
| [`chains/solana/examples/voting`](https://github.com/dwallet-labs/ika-pre-alpha/tree/main/chains/solana/examples/voting) | Example voting program: **`anchor`**, **`e2e-rust`**, **`e2e`**, **`native`**, **`pinocchio`**, **`quasar`** — **no `react`** |
| [`chains/solana/examples/multisig`](https://github.com/dwallet-labs/ika-pre-alpha/tree/main/chains/solana/examples/multisig) | Same frameworks **plus `react`** |
| [`chains/solana/examples/protocols-e2e`](https://github.com/dwallet-labs/ika-pre-alpha/tree/main/chains/solana/examples/protocols-e2e) | Rust crate **`e2e-protocols`** (`cargo run`): **gRPC-only** E2E against the **mock** server — exercises DKG curves, presign/sign schemes, imported key, `ReEncryptShare`, `MakeSharePublic`, `FutureSign`, `SignWithPartialUserSig` per file header in [`src/main.rs`](https://raw.githubusercontent.com/dwallet-labs/ika-pre-alpha/main/chains/solana/examples/protocols-e2e/src/main.rs) — **not** an on-chain voting/multisig program |

### Gap vs current skill

- [`SKILL.md`](skills/ika-solana-prealpha/SKILL.md) and [`references/frameworks.md`](skills/ika-solana-prealpha/references/frameworks.md) cover **CPI SDKs** (Pinocchio / Native / Anchor / Quasar) but **do not** point agents at **example app paths** or **`protocols-e2e`**.
- [`flows.md`](skills/ika-solana-prealpha/references/flows.md) flows 7–8 and [`instructions.md`](skills/ika-solana-prealpha/references/instructions.md) say “verify against `…/voting`” without listing **per-framework** directories or **`_shared`**.

### Recommended change (minimal, token-efficient)

Add **one** new reference file (keeps hub small per writing-skills):

- **[`references/examples.md`](skills/ika-solana-prealpha/references/examples.md)** (~80–150 words + one table):
  - Link to repo root: [examples](https://github.com/dwallet-labs/ika-pre-alpha/tree/main/chains/solana/examples).
  - **Table:** rows = `voting` / `multisig` / `protocols-e2e` / `_shared`; columns = purpose + which subdirs exist (explicit that **voting has no `react`**, **multisig has `react`**).
  - **`protocols-e2e`:** 2–3 sentences: binary name, mock gRPC default, “not Solana program CPI sample” — avoid duplicating [`grpc-api.md`](skills/ika-solana-prealpha/references/grpc-api.md).
  - **Framework row** (or second small table): `anchor` \| `native` \| `pinocchio` \| `quasar` \| `e2e` (TS) \| `e2e-rust` \| `react` (multisig only) — each cell: “`{voting|multisig}/<dir>`” pattern only.

Then **wire** (small edits):

- [`SKILL.md`](skills/ika-solana-prealpha/SKILL.md): add **one row** to the references table (“example apps + gRPC `protocols-e2e`”).
- [`flows.md`](skills/ika-solana-prealpha/references/flows.md): one line under flows 7 / 8 pointing to [`examples.md`](skills/ika-solana-prealpha/references/examples.md) for **which folder** to open for Anchor vs Quasar vs e2e.
- [`frameworks.md`](skills/ika-solana-prealpha/references/frameworks.md): after the framework comparison table, **one bullet**: “Runnable program samples: `examples.md`.”
- [`instructions.md`](skills/ika-solana-prealpha/references/instructions.md): replace vague “verify against voting” with a link to **`examples.md`** (keep “may drift” one line).

**Optional CSO** (only if under frontmatter 1024 chars): add to `description` triggers like `chains/solana/examples`, `e2e-protocols`, `protocols-e2e` — do **not** paste the whole table into YAML (writing-skills: description = when to use, not workflow).

### writing-skills alignment (Part A)

| guideline | how we satisfy it |
| --- | --- |
| **Reference skill** | New file is **index + table**, not a story |
| **Token efficiency** | Single new file; hub gains one table row + flows get one line each |
| **Discovery / keywords** | Table + path strings + optional `description` tokens |
| **No @ cross-links** | Use normal markdown links to `references/examples.md` |
| **TDD / verification** | Todo: **retrieval check** — e.g. subagent or self-check prompt: “Where is multisig Quasar sample?” answer must cite `examples.md` / path without inventing `voting/react` |

**Iron law note:** Full RED subagent runs are ideal; **minimum** acceptable verification for this edit is the structured retrieval check above plus `grep` that `voting/react` does not appear.

### Part A — out of scope (unless you expand later)

- Duplicating upstream README or per-crate build instructions — link to repo / `justfile` instead.
- Documenting every instruction byte in examples — stays in example sources + existing [`instructions.md`](skills/ika-solana-prealpha/references/instructions.md) for the **dWallet** program.

---

## Part B — `/ika-solana-prealpha audit` / `audit-force` (hybrid, OS-agnostic)

### Invocation (same idea as `i-impeccable`)

Reference: same pattern as **i-impeccable** (`SKILL.md` sections “Craft / Teach / Extract”): modes are **trailing tokens** on one skill-aligned slash, e.g. `/i-impeccable teach`, documented as “If this skill is invoked with the argument `teach`…”.

Apply the same pattern:

- **`/ika-solana-prealpha audit`** — agent loads this skill, runs **docs-revision / drift gate first**; if stale → **stop** (no user-repo audit). If fresh → `node scripts/audit-ika-solana-prealpha.mjs` + semantic checklist (flows / grpc-api / account-layouts) with file:line cites.
- **`/ika-solana-prealpha audit-force`** — **skip** the stale gate (print explicit warning that skill may be behind upstream `docs/`), then same script + checklist.

**No separate project `.cursor/commands/*.md` is required** if Cursor already exposes the skill as `/ika-solana-prealpha` with optional trailing arguments (mirrors impeccable). Routing lives in [`SKILL.md`](skills/ika-solana-prealpha/SKILL.md) as dedicated sections, like **Craft / Teach / Extract** in impeccable.

### Gate: skill freshness first (`audit` only)

1. Load [`references/docs-revision.md`](skills/ika-solana-prealpha/references/docs-revision.md) from this skill bundle.
2. Staleness per that file (tracked `docs/` vs `main` on `ika-pre-alpha`, etc.).
3. **If stale:** blocked summary + compare link + hub line about not silently rewriting → **stop** (audit mode only).
4. **`audit-force`:** document that the agent must still **surface** staleness in the report header, then proceed.

The **Node script** should implement drift + scans deterministically (`node` + stdlib); the skill text tells the agent **when** to run it and **how** to interpret exit codes / stdout.

### After the gate

- **Node (`scripts/audit-ika-solana-prealpha.mjs`):** drift (unless `--force`), deps, canonical strings vs environment table in [`SKILL.md`](skills/ika-solana-prealpha/SKILL.md).
- **Agent:** semantic checklist vs [`flows.md`](skills/ika-solana-prealpha/references/flows.md), [`grpc-api.md`](skills/ika-solana-prealpha/references/grpc-api.md), [`account-layouts.md`](skills/ika-solana-prealpha/references/account-layouts.md).

### Part B — deliverables

- [`SKILL.md`](skills/ika-solana-prealpha/SKILL.md): **Audit mode** + **Audit-force mode** sections + hub pointer to script (todos `skill-audit-modes`, `skill-hub-audit-pointer`).
- [`scripts/audit-ika-solana-prealpha.mjs`](scripts/audit-ika-solana-prealpha.mjs): stdlib-only; `--force` aligns with `audit-force`.
- **Fallback only:** if a Cursor build **does not** register `/ika-solana-prealpha` with args, add a thin [`.cursor/commands/ika-solana-prealpha.md`](.cursor/commands/ika-solana-prealpha.md) that duplicates the routing (last resort).

### Caveats (why it might not work)

| risk | mitigation |
| --- | --- |
| Slash token must match skill **`name`** in YAML (`ika-solana-prealpha`) exactly as Cursor generates it | Verify after install; hyphen/case quirks → adjust `name` or use fallback command file |
| Agent must **read** the mode sections (same as any skill) | Put **Audit mode** / **Audit-force** headings early or link from hub “when user types …” |
| writing-skills **CSO**: putting `audit` / `audit-force` in `description` helps discovery but repeats impeccable’s “call with …” pattern | Keep **one short clause** in description; full steps stay in body |
| `audit-force` is easy to misuse | Require printed **warning** + stale diff summary before continuing |

### Optional product choice

`audit-force` is the escape hatch; no separate `--force` UX required beyond script flag for the same behavior.

---

## Suggested execution order

1. Part A (`add-examples-md` → `wire-crossrefs` → `optional-description-cso` → `verify-writing-skills`) so the skill index is correct before you point audits at it.
2. Part B (`node-script`; `skill-audit-modes` + `skill-hub-audit-pointer`; optional fallback `.cursor/commands` only if `/ika-solana-prealpha` args are not available).

**Note:** Retired duplicate: `%USERPROFILE%\.cursor\plans\audit-ika-solana-prealpha_command_48355162.plan.md` — use this merged file.
