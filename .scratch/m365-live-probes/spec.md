# M365 Live Probes

**Status:** Backlog — all probes need a live, rested M365 account (laptop)
**Source:** `docs/hypotheses.md` §7 probe backlog (ordered by info-gain ÷ cost)

## What this effort is

The reverse-engineering probe backlog: small, single-purpose probes that
measure one M365 behaviour at a time. Discipline (from AGENTS.md):

- **One thread at a time.** Throttle tracks threads-started; never fire
  concurrent requests, never loop fresh conversations back-to-back. Space
  runs with cooldowns.
- **Rested account only.** Run comparative probes on a rested account —
  degradation makes `Disengaged` look like a format/prompt failure.
- **n=1 is noise.** Confirm winners with `--repeat`; rotate order across
  runs.
- **Record.** Update `docs/hypotheses.md` with sample size + evidence
  pointer; promote conclusive findings to `docs/m365-copilot-api.md`.

## Probe plumbing

`scripts/_probe-chat.mjs` — one M365 turn in → structured result out.
Copy an existing probe rather than starting from scratch. All probes are
`.mjs` under `scripts/`; run with
`M365_NO_INTERACTIVE=1 CHROMIUM_PATH=$(which chromium) bun scripts/<probe>.mjs`.

## Ticket map

| # | Probe | Script | Blocked by |
|---|-------|--------|------------|
| 01 | Disengaged calibration (dea threshold) | 🔴 unwritten | laptop + rested account |
| 02 | Tool-compliance statistical (`--repeat 5`) | 🟢 exists | rested account, ~150 msgs |
| 03 | Usage-endpoint hunt v2 (full browser headers) | 🔴 unwritten | laptop (0 msgs — GETs) |
| 04 | InputMethod / experienceType flip | 🔴 unwritten | laptop, ~5 msgs |
| 05 | Tone comparison (F2–F4 off `magic`) | 🔴 unwritten | laptop, ~50 msgs |
| 06 | ConversationTransferToken migration | 🔴 unwritten | laptop, ~5 msgs |
| 07 | Admin-portal usage dig (Playwright) | 🔴 unwritten | laptop (0 msgs — UI) |
| 08 | Run the five 🟢 green probes | 🟢 exists | laptop + rested account |