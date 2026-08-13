# Plan: 02 — Tool-compliance statistical run
> Ticket: .scratch/m365-live-probes/issues/02-tool-compliance-repeat.md · Status: ready-for-agent · Blocked by: none
## Purpose
Turn F2's "10% faster" (single n=1 run, 30 cells) into a real comparison with error bars via `--repeat 5`. Produces the SOLVED-rate/outcome-mix baseline that decides whether the fenced-format/shell-routing win (F2–F4) is real or noise — load-bearing for the usable-agent-in-pi goal.
## Preconditions
- Explicit user authorization for live M365 probes; rested account (backoff level 0, ≥24h rest, fresh cache 2026-08-10 verified).
- Cost: 6 variants × 5 prompts × 5 reps = 150 fresh conversations → at ≤12/hr with ≥3 min spacing this spans multiple days; plan windows accordingly.
- No proxy build needed; standalone script under bun.
## Steps
1. Authorize; confirm rested account (backoff 0).
2. Ensure order rotation in `scripts/tool-compliance-experiment.mjs`: prompt/variant order must differ across the 5 reps (add a `--shuffle` or per-rep rotation if absent — current script has none).
3. Run: `M365_NO_INTERACTIVE=1 CHROMIUM_PATH=$(which chromium) bun scripts/tool-compliance-experiment.mjs --repeat 5` (keep the agent path default; use `--no-agent` only if a condition requires it, and record it). Strictly sequential; ≥3 min between conversations; hard stop at first empty-503/at-limit; resume in the next rested window rather than pushing.
4. From `scripts/tool-compliance-out/<TS>/results.json` scoreboard, extract per condition: SOLVED rate (`good/n`), outcome mix (verdicts), median/p95 latency, dea_violation median — with n = 5 per cell.
5. Update F2–F4 verdicts in `docs/hypotheses.md`: confirmed / refuted / inconclusive, each with n and effect size vs the noise floor; append ## Comments to the ticket.
## Acceptance
- Run on a rested account with order rotated across runs (rotation verified in script or run protocol).
- SOLVED rate + outcome mix per condition reported with n and error bars (not point estimates).
- F2–F4 verdicts updated in `docs/hypotheses.md` with n.
## Evidence
- `scripts/tool-compliance-out/<TS>/results.json` (raw, per-rep) + scoreboard; `docs/hypotheses.md` F2–F4; ticket ## Comments.
## Risks
- 150 msgs over multiple days: account state drifts between windows — log backoff level per run; split runs are pooled only if no throttle evidence. Compliance is prompt-shape-sensitive (crafted vs agentic divergence known) — report per-prompt breakdown, not just pooled. Empty reply without Disengaged frame = thread throttle, not content filter. n=5 gives ±~20% resolution on rates; don't over-read sub-noise deltas.
