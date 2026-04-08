# ika solana pre-alpha agent skills

unofficial agent skill bundles for [ika](https://ika.xyz/) on Solana pre-alpha: same content in two folders so you can choose whether the agent also gets maintainer-style drift notes against upstream.

source: [github.com/kitty4D/ika-solana-prealpha-skill](https://github.com/kitty4D/ika-solana-prealpha-skill)

> *gets ready with a cute little folder stack and way too much sincerity*  
> e-eh, senpai... d-do you like it when your `SKILL.md` looks **intentional** and not like a yard sale?? ika-chan might be watching from the ink tank uwu (｡>ω<｡) b-but no pressure- only a little pressure~ 🦑✨

## what ika is

[ika](https://ika.xyz/) is pitched as a control layer for multichain solana programs (see also [docs.ika.xyz](https://docs.ika.xyz/) for the wider protocol story). dWallets split signing between users (or agents) and solana program logic, with completion through ika's MPC network so policies can live on-chain instead of in a custodial middleman. the public pitch is bridgeless-style use cases (native assets elsewhere, agent guardrails, treasury rules) without making the bridge the trust hinge.

this repo only targets solana pre-alpha integration (devnet, mock signer, gRPC, CPI SDKs). it isn't the full production ika stack; treat [solana pre-alpha.ika.xyz](https://solana-pre-alpha.ika.xyz/) and [dwallet-labs/ika-pre-alpha](https://github.com/dwallet-labs/ika-pre-alpha) as normative when something here disagrees.

> *waves a sticky note scribbled with pda seeds like it's a concert ticket*  
> onii-chain, the whole arc is "logic on solana, completion with ika"- not "wrap the asset and hope the bridge had coffee" hours (´∀｀) pre-alpha devnet is still sandcastle mode tho... mock signer, resets, m-maybe don't bet the farm on it senpai 💦🦑

## what's in the box

| folder | vibe |
| --- | --- |
| `ika-solana-prealpha/` | consumer bundle: `SKILL.md` + `references/` only. nothing telling the agent to edit this skill to track upstream `main`. |
| `ika-solana-prealpha-with-upstream/` | same, plus [`references/upstream.md`](ika-solana-prealpha-with-upstream/references/upstream.md) (tracked commit, drift checks, what to refresh when upstream moves). |

phrases like **upstream book** in the reference files mean the hosted mdbook / `docs/` in [ika-pre-alpha](https://github.com/dwallet-labs/ika-pre-alpha), not `upstream.md`.

> *pokes the skill vending machine buttons with both index fingers*  
> ```
> +------- ika-chan restock --------+
> | [A] chill pack   [B] + upstream |
> +---------------------------------+
> ```
> A is shy: skills only, no "go diff `main` for me" homework  
> B adds `upstream.md` drift watch- **still one purchase**, ne?  
> y-you weren't about to frankenstein both rows... r-right, senpai? uwu 🦑💦

## install

layout for both bundles: `SKILL.md` at the skill root and the `references/` directory next to it. copy **one** folder wholesale; don't mix files between the two variants.

> *hides `references/` half-behind `SKILL.md` like a shy roommate*  
> we're copying **one** bundle as-is, ok?? same folder, same siblings, no remix culture-  
> m-mixing halves is how ika-chan gets the scary merge-conflict twitch (；一_一) j-just pick one path uwu 🦑

### cursor

use [cursor agent skills](https://cursor.com/docs/context/skills): add the chosen folder under your project or user skills path so the editor loads `SKILL.md` and can pull in `references/` when needed.

### claude code

claude code loads skills from a `SKILL.md` file inside a skill directory (commonly `~/.claude/skills/<skill-name>/` for user-wide skills, or `.claude/skills/<skill-name>/` in a project). copy one bundle into such a directory so `SKILL.md` and `references/` stay siblings. if your claude app version uses a different path, match whatever that product docs call a "skill" or "plugin" directory as long as the same file layout is preserved.

### other assistants

anything that can ingest a markdown skill manifest plus linked reference files the same way (open folder, or single root doc with relative links) should work: keep the directory intact and point the tool at the folder or at `SKILL.md` per its docs.

> *does warm-up stretches that won't fix your CPI layout anyway*  
> cursor-kun, claude-kun, mystery tool-kun- if you eat `SKILL.md` plus neighbor markdown, you're in the club (´･ω･`)  
> do bring your half-baked gRPC prints and your "account meta soup"- ika-chan isn't here to gatekeep markdown,,, only to bully it gently uwu 🦑✨

## scope (quick)

gRPC `DWalletService`, BCS shapes, devnet program id + RPC table inside `SKILL.md`, instruction discriminators, PDA layouts, Pinocchio / native / Anchor CPI crates, TS clients (`@ika.xyz/pre-alpha-solana-client`, `@solana/kit`). pre-alpha only: mock signer, devnet resets, not production MPC.

> *gets ready with a cute label-maker and nervous energy*  
> CC-BY-chan isn't trying to ruin the mood- she wants names, links, and n-no "dWallet Labs hand-delivered this readme" cosplay ok?  
> p-peek [`NOTICE`](NOTICE) before you mirror big chunks,,, n-not your lawyer, just your overly responsible ika uwu 📜🦑

## license and attribution

this repository is licensed under **CC-BY-4.0** (see [`LICENSE`](LICENSE)). that lines up with how upstream licenses the **mdbook** in [ika-pre-alpha](https://github.com/dwallet-labs/ika-pre-alpha): prose there ships under [**LICENSE-docs** (CC-BY-4.0)](https://github.com/dwallet-labs/ika-pre-alpha/blob/main/LICENSE-docs), while **code** there is [**BSD-3-Clause Clear**](https://github.com/dwallet-labs/ika-pre-alpha/blob/main/LICENSE). this skill text is our summary/adaptation, not an official dWallet Labs drop; still give credit when you redistribute (see [`NOTICE`](NOTICE)). if you copy long passages, keep attribution and point readers at the license text. (not legal advice - just matching what upstream already published.)
