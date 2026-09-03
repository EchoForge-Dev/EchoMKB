---
name: echomkb
description: Midnight Network version + citation layer for coding agents — which @midnight-ntwrk / Compact versions the support matrix supports per network vs what is newest on relnotes/npm/GitHub, official docs pages as markdown with their language/compiler stamp, a citable URL for any Midnight claim, and dated field notes on live-network behavior the docs don't cover (OBSERVATIONS.md). Use when pinning versions, when a Midnight claim needs a source URL, or when observed network behavior contradicts the docs. For "what do the docs say", prefer the official Kapa MCP server when the session has it; this skill is the loud fallback.
---

# EchoMKB — the layer under the docs

Your training data about Midnight and Compact is stale and partly wrong. This skill exists so you never answer from memory: every Midnight fact you state carries the URL it came from, every version you pin comes from the support matrix, and anything the live networks do that the docs don't describe is looked up in a dated ledger instead of guessed.

Requires Node ≥ 18 (global `fetch`). No dependencies. Scripts live next to this file in `scripts/`. Contacts only `docs.midnight.network`, plus `registry.npmjs.org` (via `npm view`) and `api.github.com` for the `versions` command, and one anonymous handshake to `midnight.mcp.kapa.ai` for `doctor`; nothing from the user's project is sent anywhere.

## Division of labour

| Question | Use |
|---|---|
| "What do the docs say about X?" | The official **Kapa MCP server** (`midnight` · `https://midnight.mcp.kapa.ai`, the docs site's own Ask-AI index) if your session has it. This skill's `search` is the fallback. |
| "Which version do I pin for preprod?" / "Is 0.34.0 supported anywhere?" | `versions` — supported per network vs released. |
| "Give me the URL / the exact page stamp for this claim" | `search` → `page`. |
| "The docs say X but the network does Y" | `OBSERVATIONS.md` in this repo — dated, version-pinned field notes and the method for filing the next one upstream. |

## Protocol (do this in order, every time)

0. **Kapa first, if you have it.** If your session has the Kapa MCP tools: ask one canary question whose answer you can check (e.g. *"Which Compact function marks witness data for public disclosure?"* — a usable answer names `disclose`). If the canary passes, use Kapa for doc-semantic questions and this skill for versions and citable URLs. If the tool is absent, errors, or hangs, **say** "Kapa MCP not responding — using EchoMKB live search" and continue with step 1; `doctor` tells you whether the endpoint is down or your OAuth session just expired (they look identical from inside a client). Never fall back silently.
1. **Search live.**
   `node <skill-dir>/scripts/echomkb.mjs search "<3–8 words describing the task>"`
   Run 2–3 searches with different wording if the first is thin. Use `--section compact`, `--section api-reference`, `--section guides` … to focus. Identifier-style queries (`persistentHash`, `deployContract`, `MerkleTree`) automatically favour API reference pages.
2. **Read the whole page before relying on it.**
   `node <skill-dir>/scripts/echomkb.mjs page <url>` — excerpts are for triage, not for quoting signatures or syntax.
3. **For anything involving versions**, run
   `node <skill-dir>/scripts/echomkb.mjs versions`
   and report *both* columns: what the support matrix **supports** on the target network and what is **released**. The matrix lagging the newest release is the normal state, not a warning — the newest version is usually the one you must *not* pin yet. Deployments follow the matrix; compiler ↔ runtime must match exactly; pin `@midnight-ntwrk/*` without `^`/`~`. The command also prints the docs' own page stamp (language / compiler) next to the supported toolchain: when the docs are written for a newer compiler than any network supports, say so before quoting syntax.
4. **Cite.** Every Midnight-specific claim in your answer names its source URL. If you could not find a page that states the fact, say so instead of inferring.
5. **Verify code.** Compact and SDK code is only "correct" after it compiles (`compact compile`) and, where possible, runs. Present unverified code as unverified.
6. **When behavior contradicts the docs**, open `OBSERVATIONS.md` (repo root) before debugging from scratch: the symptom may already be there with a workaround and an upstream issue. If it is not, follow the method section at the top of that file (pin the environment quadruple → reproduce on a second network → search existing issues → report upstream) and propose a new entry. You may draft the upstream issue and file it yourself (`gh issue create` on `midnightntwrk/*` or `input-output-hk/lace`), **but only after the human has read the final text and explicitly authorized filing** — never on your own initiative; the issue is submitted under their name.

## When the network is down

`node <skill-dir>/scripts/echomkb.mjs doctor` tells you. There is no bundled snapshot — a snapshot rots the day it is taken. The tool may serve a page it fetched earlier from its own temp-dir cache, and it labels that output `stale-cache` with the age; if you use it, **say so in the answer**. If nothing is cached, say the docs are unreachable. Never answer a Midnight question from memory because the network was down.

## Commands

| Command | What it does |
|---|---|
| `search <query> [--max 8] [--fetch 3] [--section s] [--json]` | Rank pages from the live `llms.txt` index (≈1,300 pages), open the top hits, print heading-anchored excerpts with URLs and the page's own "Compact language X, compiler Y" stamp when present. |
| `page <path-or-url> [--max-chars N]` | Full page as markdown (`.md` endpoint; strips HTML if a page has none). |
| `versions [--no-npm] [--no-github] [--json]` | Support matrix per network (supported) vs relnotes / npm / GitHub (released), with a status column and the docs page stamp vs the supported toolchain. |
| `index` | Sections of the live docs index with counts — orient yourself before searching. |
| `doctor` | Connectivity + cache check, plus a Kapa MCP endpoint probe that separates "endpoint down" from "your OAuth session is missing/expired" (cache in the OS temp dir, 15 min for the index, 60 min for pages; `--fresh` bypasses). |

## Hard rules

- Live docs > this tool's own labelled cache > nothing. Never anything recalled from training data. On conflict between two pages, the support matrix wins for versions; mention the conflict.
- The support matrix page is newer than per-component release-notes pages when they disagree; GitHub tags and npm are usually *ahead* of both (released, not yet supported).
- Docs pages carry a stamp like "Compact language version 0.26.0, compiler version 0.34.0" — repeat it when quoting syntax, because syntax changes between language versions and the docs may be ahead of what the networks support.
- Do not invent package names, CLI flags, stdlib signatures, or disclosure rules. If the docs do not show it, you do not know it.
- Privacy is the product: default to the minimal on-chain footprint (data minimisation) when proposing designs, and flag every `disclose()`.

## Provenance

Built by EchoForge (m.echoforgeef.com/echomkb). Source: https://github.com/EchoForge-Dev/EchoMKB · Apache-2.0.
