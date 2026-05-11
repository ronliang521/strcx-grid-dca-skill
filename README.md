# STRCx Solana Grid DCA Skill

This repo contains a single Cursor skill: a **Solana ladder/grid DCA** for mint:

`Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH`

The token is often shown as **STRCx** in OKX/onchainOS.

## What it does

- **Buy grid**: when OKX USD price \(P\) falls to **≤ 102**, it buys **50 USDT** per **0.5 USD** step down: \(B_n = 102 - 0.5n\), until USDT is insufficient.
- **Sell grid**: after price rises to **> 120**, it sells **~50 USDT worth** per **0.5 USD** step up: \(S_m = 120 + 0.5m\), until STRCx is depleted.
- Uses **onchainOS** (`onchainos`) as the primary data source and execution tool.
- Persists rung progress locally (state file) to avoid double-filling the same rung.
- Includes a **contest mode** section for the OKX Agentic trading competition (compliance-focused; no wash trading guidance).

## Install into Cursor (project skill)

Copy the folder into your project:

- `ron-dca-crclx-solana-ladder/` → `.cursor/skills/ron-dca-crclx-solana-ladder/`

Then in Cursor, reference the skill by name or trigger terms (see `SKILL.md`).

## Files

- `ron-dca-crclx-solana-ladder/SKILL.md`
- `ron-dca-crclx-solana-ladder/reference.md`

