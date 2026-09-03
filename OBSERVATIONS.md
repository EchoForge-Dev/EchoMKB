# Observations

What the live Midnight networks actually did, where that differs from what the documentation describes. **Observations, not facts** — every entry is pinned to a date and exact versions, and newer releases may have changed anything below. For what the docs say, use the [official docs](https://docs.midnight.network) or Kapa; this file records the layer underneath, the one you hit when the docs' answer doesn't match what your terminal shows.

Seeded 2026-08-31 from the MLH Midnight Hackathon weekend ([EchoCert on Midnight](https://github.com/EchoForge-Dev/EchoCert_Midnight_Demo), 2026-08-21 → 08-30). New entries go on top.

## Method

How each entry was produced, and the checklist for producing the next one when behavior contradicts the docs:

1. **Pin the environment quadruple** before anything else: network · node/indexer versions · wallet-SDK + dust-wallet versions · whether the failing circuit *reads* or *writes* ledger state.
2. **Reproduce on a second network.** When the *same* asymmetry shows up on two different networks, suspect the mechanism they share before the code that differs.
3. **Search existing issues first** — `midnightntwrk/*` and `input-output-hk/lace`. Read what sits above your entry; two of the four seeds below already existed since May.
4. **Report upstream** using the skeleton of [BUGREPORT.md](https://github.com/EchoForge-Dev/EchoCert_Midnight_Demo/blob/main/BUGREPORT.md): status · pinned environment · reproduction · evidence · how it was misdiagnosed · what the SDK should do · open questions. An agent may draft the issue and submit it (e.g. with `gh issue create`), but only after a human has read the final text and explicitly said to file it — the issue carries the human's name. wallet#700 and lace#2257 below were filed exactly this way.

Debugging order that held up in practice: **sync state first** (shielded / unshielded / dust progress and completeness), then registration, then transactions. Every misdiagnosis of that weekend started from not being able to see sync state.

---

## Ledger

### [lace#2257](https://github.com/input-output-hk/lace/issues/2257) · Lace initial Preprod sync dies at ~99% every cycle

- **Symptom:** first-time Preprod sync reaches ~99% after 26–39 min, then the extension relocks or crashes and resumes from an old checkpoint (measured 99% → 46%, then 99% → 54%). A fresh wallet never reaches 100%, so "Generate tDUST" can never succeed. During sync the UI is sluggish — network/proof-server/signing dialogs need repeated clicks.
- **Trigger:** initial (cold) sync, ~60–70 min of work; extension memory climbs to ~3.2 GB at 51% and ~4.0 GB at 54%.
- **Environment:** Lace 2.2.3 · Chrome 143 (arm64, macOS) · Preprod.
- **Best guess:** extension worker hits a memory ceiling near 4 GB and is killed; sync restarts from the last checkpoint and dies again in the same zone.
- **Workaround:** none inside Lace. Anything that must actually finish went headless (server-side wallet SDK with persisted checkpoints).
- **Observed:** 2026-08-30, two controlled runs. Filed the same day.
- **Status:** OPEN (filed by me, 2026-08-30).

### [midnight-wallet#436](https://github.com/midnightntwrk/midnight-wallet/issues/436) · Preprod dust event 565975 crashes wallet sync on older dust-wallet

- **Symptom:** dust-wallet sync on Preprod crashes deterministically at dust ledger event 565975 (`midnight:event[v9]` tag) and never reaches tip; DUST submissions then fail with `1010 Custom error: 170` (InvalidDustSpendProof).
- **Trigger:** syncing Preprod past that event with `dust-wallet` 4.0.0 (also reproduced upstream on the support-matrix 3.0.0).
- **Environment (upstream report):** Preprod · public RPC + indexer v4 · dust-wallet 4.0.0 / 3.0.0.
- **Observed workaround:** my stack on `dust-wallet` 4.2.0 (wallet-sdk 1.2.0) synced through the same chain without incident, 2026-08-28 → 30. Not a confirmed fix, an observation.
- **Status:** OPEN upstream since 2026-05-22 (reported via Discord #dev-chat). Matched to a stuck developer in #mlh-hackers on 2026-08-30.

### [midnight-wallet#415](https://github.com/midnightntwrk/midnight-wallet/issues/415) · Registering for DUST right after the faucet is rejected (error 138)

- **Symptom:** a brand-new wallet that calls `registerNightUtxosForDustGeneration` immediately after receiving tNIGHT gets `1010: Invalid Transaction: Custom error: 138` (`BalanceCheckOverspend`); retrying minutes later, the identical-shape transaction is accepted. A faucet-then-register script fails deterministically on the first run.
- **Root cause (per upstream analysis):** the registration's fee budget is projected from DUST "generated so far" by the just-received UTXO — one or two blocks after receipt that projection sits right at the edge of the registration's own fee.
- **Workaround:** wait a few minutes between receiving NIGHT and registering. A script wallet that fully syncs first avoids it naturally. In Lace this surfaces as a silently vanished transaction (no history entry) when Generate tDUST is clicked immediately after the faucet.
- **Environment:** undeployed (local devnet) per upstream; symptom matched on Preprod via Lace, 2026-08-23 → 30.
- **Status:** OPEN upstream since 2026-05-18. Matched during #mlh-hackers triage on 2026-08-30.

### [midnight-wallet#700](https://github.com/midnightntwrk/midnight-wallet/issues/700) · Read-only circuit → zero fee → node rejects or wallet panics

- **Symptom:** deploy and state-*writing* circuits land every time; a state-*reading* circuit call fails every time. Pre-GA node rejects in ~2 s (`1010 Custom error: 117`, NotNormalized); GA Preprod hangs 21–25 min, then the wallet panics inside the fee dry-run — `(FiberFailure) Wallet.Other: unreachable` (WASM `RuntimeError: unreachable`).
- **Trigger:** a circuit that only reads ledger state, on a quiet chain → wallet computes fee = 0 → dust wallet balances with an empty `DustActions` → transaction is not normalized.
- **Environment:** local devnet (node 0.22.5, indexer 4.2.1) **and** Preprod (node 1.0.1 and 1.0.2, indexer 4.3.3-hotfix) · wallet-sdk 1.2.0 / dust-wallet 4.2.0 · midnight-js 4.1.1 · Compact toolchain 0.31.1 (language 0.23.0) · read-only circuit. 8/8 reproductions before the fix.
- **Workaround (verified):** `DEFAULT_DUST_OPTIONS.additionalFeeOverhead = 1_000_000n` (any positive value) — the identical transaction then lands on both networks in ~19 s ([Subscan 2306120-3](https://midnight-preprod.subscan.io/extrinsic/2306120-3)).
- **Open question:** why the GA path *panics* where the pre-GA node merely *rejects* — plausibly a zero-fee division/unwrap reached only with live-network ledger parameters. Adjacent balancing-loop issue: [#438](https://github.com/midnightntwrk/midnight-wallet/issues/438).
- **Observed:** 2026-08-21 → 08-28 (misdiagnosed as a historic-Merkle-root problem for a week; the devnet reproduction broke the wrong theory in one error code). Full write-up: [BUGREPORT.md](https://github.com/EchoForge-Dev/EchoCert_Midnight_Demo/blob/main/BUGREPORT.md).
- **Status:** OPEN (filed by me, 2026-08-30).

---

*Format per entry: symptom · trigger · environment quadruple · root cause or best guess · workaround · upstream link · observation date · status. Corrections and new observations welcome — open an issue on this repo.*

Apache-2.0 © 2026 EchoForge
