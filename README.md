# EchoMKB

The layer under the Midnight docs, for coding agents — **which versions each network actually supports**, **a citable URL for every claim**, and **the loop back upstream** when the network does something the docs don't say: search the trackers first, report after the human approves. Everything is fetched at answer time; nothing is bundled, nothing is recalled. If a source is unreachable the agent says so: rather no answer than a wrong one.

> [!NOTE]
> This project extends the Midnight Network with additional developer tooling.

```sh
npx skills add EchoForge-Dev/EchoMKB     # install
npx skills update echomkb                # later: pull the latest
```

Works with every agent the [skills CLI](https://github.com/vercel-labs/skills) supports (Claude Code, Cursor, Codex, Copilot, Gemini CLI, …). Node ≥ 18, zero dependencies.

**v0.2 (2026-09-03).** Repositioned after the docs maintainers' review: the official [Kapa MCP server](https://docs.midnight.network/ai-integration/kapa-mcp-server) answers *what the docs say*; EchoMKB now hands that question to Kapa when the session has it, and keeps the parts Kapa does not do — supported-vs-released versions per network, page stamps and URLs, a probe that tells you when Kapa is silently dead, and the reporting step. The bundled snapshot is gone (it rotted, as predicted).

**One loop.** *Find answers* — [Kapa MCP](https://docs.midnight.network/ai-integration/kapa-mcp-server) · *Build it* — [Midnight Expert](https://docs.midnight.network/ai-integration/midnight-expert) · *Go further* — community skills such as [Midnight-skills](https://github.com/Kali-Decoder/Midnight-skills) · *Report it* — EchoMKB. What the live networks do that the docs don't say goes back upstream as an issue, so the next answer Kapa gives is better.

## What it does

| | |
|---|---|
| `search <query>` | live `llms.txt` index (≈1,300 pages) → ranked pages → opened → heading-anchored excerpts with URLs |
| `page <url>` | one docs page as markdown, with its "language X, compiler Y" stamp |
| `versions` | support matrix per network (**supported**) vs relnotes / npm / GitHub (**released**) — and the docs page stamp vs the supported toolchain |
| `issues <words>` | live GitHub issue search over the upstream trackers (`midnightntwrk/*`, `input-output-hk/lace`, `LFDT-Minokawa/compact`) — run it before debugging from scratch and before filing a duplicate |
| `doctor` | connectivity + Kapa MCP endpoint probe: is it down, or is your OAuth session dead? (the two look identical from inside an MCP client) |
| `index` | sections of the live docs index |

The skill's protocol (in [`skills/echomkb/SKILL.md`](skills/echomkb/SKILL.md)) makes the agent try Kapa first and fall back *out loud*, read whole pages before quoting, cite every claim, and report both "supported" and "released" on version questions — the matrix lagging the newest release is the normal state, not a warning.

**Report it** is the step the other tools stop before. When the network does something the docs don't describe, the protocol has the agent search the trackers, pin the environment, reproduce on a second network, and write a report the maintainers can use — then file it, or comment on the closest existing issue, only after you have read the final text and said so. The method came out of two such reports from the MLH Midnight Hackathon ([midnight-wallet#700](https://github.com/midnightntwrk/midnight-wallet/issues/700), [lace#2257](https://github.com/input-output-hk/lace/issues/2257)); the write-up that shaped it is [BUGREPORT.md](https://github.com/EchoForge-Dev/EchoCert_Midnight_Demo/blob/main/BUGREPORT.md).

Try it without installing:

```sh
node skills/echomkb/scripts/echomkb.mjs versions
node skills/echomkb/scripts/echomkb.mjs doctor
node skills/echomkb/scripts/echomkb.mjs issues "read-only circuit zero fee"
node skills/echomkb/scripts/echomkb.mjs search "disclose witness ledger write"
```

## Privacy

Only `docs.midnight.network` is contacted for search/page; `versions` additionally runs `npm view` and calls `api.github.com`; `issues` calls `api.github.com` only; `doctor` additionally sends one anonymous MCP handshake to `midnight.mcp.kapa.ai` (no credentials, no content). Nothing from your repository leaves your machine.

## Credits

Built on the official Midnight documentation, which exposes [`llms.txt`](https://docs.midnight.network/llms.txt) and per-page `.md` endpoints, and next to the official Kapa MCP server, which this skill defers to for documentation questions. Installation is handled by the [skills CLI](https://github.com/vercel-labs/skills) from Vercel Labs.

## Links

Intro page: https://m.echoforgeef.com/echomkb · Human-readable KB: https://m.echoforgeef.com/kb · EchoForge: https://echoforgeef.com

Apache-2.0 © 2026 EchoForge
