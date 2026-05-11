# STRCx / CRCLx Solana Grid DCA Skill

This repo contains a single Cursor skill: a **Solana ladder/grid DCA** for mint:

`Xs78JED6PFZxWc2wCEPspZW9kL3Se5J7L5TChKgsidH`

The token is often shown as **STRCx** in OKX/onchainOS; the user may refer to it as **CRCLx**.

## What it does

- When OKX USD price \(P\) is **≤ 100**, it buys **50 USD** notional per **0.5 USD** step down the grid: \(L_n = 100 - 0.5n\).
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

