# EchoMKB

The layer under the Midnight docs, for coding agents — **which versions each network actually supports**, **a citable URL for every claim**, and **what the live networks do that the docs don't say**. Everything is fetched at answer time from [docs.midnight.network](https://docs.midnight.network); nothing is bundled, nothing is recalled.

> [!NOTE]
> This project extends the Midnight Network with additional developer tooling.

```sh
npx skills add EchoForge-Dev/EchoMKB
```

Works with every agent the [skills CLI](https://github.com/vercel-labs/skills) supports (Claude Code, Cursor, Codex, Copilot, Gemini CLI, …). Node ≥ 18, zero dependencies.

**v0.2 (2026-09-03).** Repositioned after the docs maintainers' review: the official [Kapa MCP server](https://docs.midnight.network/ai-integration/kapa-mcp-server) answers *what the docs say*; EchoMKB now hands that question to Kapa when the session has it, and keeps the parts Kapa does not do — supported-vs-released versions per network, page stamps and URLs, a probe that tells you when Kapa is silently dead, and a dated ledger of live-network behavior. The bundled snapshot is gone (it rotted, as predicted).

## What it does

| | |
|---|---|
| `search <query>` | live `llms.txt` index (≈1,300 pages) → ranked pages → opened → heading-anchored excerpts with URLs |
| `page <url>` | one docs page as markdown, with its "language X, compiler Y" stamp |
| `versions` | support matrix per network (**supported**) vs relnotes / npm / GitHub (**released**) — and the docs page stamp vs the supported toolchain |
| `doctor` | connectivity + Kapa MCP endpoint probe: is it down, or is your OAuth session dead? (the two look identical from inside an MCP client) |
| `index` | sections of the live docs index |

The skill's protocol (in [`skills/echomkb/SKILL.md`](skills/echomkb/SKILL.md)) makes the agent try Kapa first and fall back *out loud*, read whole pages before quoting, cite every claim, and report both "supported" and "released" on version questions — the matrix lagging the newest release is the normal state, not a warning.

What the docs *can't* answer — observed live-network behavior, dated and version-pinned, plus the method for filing the next one upstream — is in [OBSERVATIONS.md](OBSERVATIONS.md).

Try it without installing:

```sh
node skills/echomkb/scripts/echomkb.mjs versions
node skills/echomkb/scripts/echomkb.mjs doctor
node skills/echomkb/scripts/echomkb.mjs search "disclose witness ledger write"
```

## Privacy

Only `docs.midnight.network` is contacted for search/page; `versions` additionally runs `npm view` and calls `api.github.com`; `doctor` additionally sends one anonymous MCP handshake to `midnight.mcp.kapa.ai` (no credentials, no content). Nothing from your repository leaves your machine.

## Credits

Built on the official Midnight documentation, which exposes [`llms.txt`](https://docs.midnight.network/llms.txt) and per-page `.md` endpoints, and next to the official Kapa MCP server, which this skill defers to for documentation questions. Installation is handled by the [skills CLI](https://github.com/vercel-labs/skills) from Vercel Labs.

## Links

Intro page: https://m.echoforgeef.com/echomkb · Human-readable KB: https://m.echoforgeef.com/kb · EchoForge: https://echoforgeef.com

Apache-2.0 © 2026 EchoForge
