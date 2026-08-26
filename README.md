# EchoMKB

Midnight Network knowledge for coding agents — searched **live** from [docs.midnight.network](https://docs.midnight.network) on every use, with cited URLs and a version-drift report.

```sh
npx skills add Charlie0113-T/EchoMKB
```

Works with every agent the [skills CLI](https://github.com/vercel-labs/skills) supports (Claude Code, Cursor, Codex, Copilot, Gemini CLI, …). Node ≥ 18, zero dependencies.

## What it does

| | |
|---|---|
| `search <query>` | live `llms.txt` index (≈1,300 pages) → ranked pages → opened → heading-anchored excerpts with URLs |
| `page <url>` | one docs page as markdown |
| `versions` | support matrix (tested) vs release notes / npm / GitHub (latest) — drift column |
| `index` · `doctor` | orientation · connectivity |

The skill's protocol (in [`skills/echomkb/SKILL.md`](skills/echomkb/SKILL.md)) forces the agent to search before answering, read whole pages before quoting, cite every claim, and report both "tested" and "newest" on version questions. Offline fallback is a dated English snapshot of EchoForge's MIDNIGHT_KB, and the agent must say when it is used.

Try it without installing:

```sh
node skills/echomkb/scripts/echomkb.mjs search "disclose witness ledger write"
node skills/echomkb/scripts/echomkb.mjs versions
```

## Privacy

Only `docs.midnight.network` is contacted for search/page; `versions` additionally runs `npm view` and calls `api.github.com`. Nothing from your repository leaves your machine.

## Links

Intro page: https://m.echoforgeef.com/echomkb · Human-readable KB: https://m.echoforgeef.com/kb · EchoForge: https://echoforgeef.com

MIT © 2026 EchoForge
