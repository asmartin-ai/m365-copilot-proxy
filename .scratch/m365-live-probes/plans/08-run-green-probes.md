# Plan: Run the green probes
> Ticket: .scratch/m365-live-probes/issues/08-run-green-probes.md · Status: ready-for-agent · Blocked by: none

## Purpose
Execute four of the five 🟢 cheap probes from `docs/hypotheses.md` §7 on a
rested account (the fifth — tool-compliance `--repeat 5` — is owned by
ticket 02; not part of this batch). (fresh MSAL cache 2026-08-10, backoff level 0). The scripts exist
but their June-9 runs predate the rested account; re-running removes
n=1-noise and throttle-state confounds so F2–F6 get real sample sizes.

## Preconditions
- Explicit user authorization for live M365 probes (standing rule).
- Rested account verified: backoff level 0; confirm with one clean turn
  before the batch (F24: back-to-back fresh threads self-throttle).
- Strictly sequential, one thread at a time; ≤12 fresh convs/hr, ≥3 min
  spacing; hard stop at first empty-503/at-limit (empty reply WITHOUT a
  Disengaged frame = thread throttle, not a verdict).
- Runners per script header: `usage-endpoint-hunt.mjs` and
  `variants-bisect.mjs` under `node`; `frame-dump-*.mjs` and
  `tool-compliance-experiment.mjs` under Bun. All with `M365_NO_INTERACTIVE=1`.

## Steps
1. `M365_NO_INTERACTIVE=1 node scripts/usage-endpoint-hunt.mjs` — 0 msgs
   (GETs only). Re-checks F5's "correctly empty" reading on the fresh
   account (v2, 2026-08-11, already falsified browser-header gating).
2. `M365_NO_INTERACTIVE=1 node scripts/variants-bisect.mjs --target
   disengaged|streaming` — ~10 msgs per target, one target per run, target
   order rotated across runs (feeds ticket 05 tone work).
3. `M365_NO_INTERACTIVE=1 bun scripts/frame-dump-probe.mjs` — 1 msg; dump
   every WS frame key; diff `keys-summary.json` against the June-9 baseline
   for newly-added fields (catches token/usage-shaped additions).
4. `M365_NO_INTERACTIVE=1 bun scripts/frame-dump-disengage.mjs` — 1 msg;
   Disengage-shaped probe (12 tools + jailbreak framing) for F6 scores.
5. ~~tool-compliance-experiment --repeat 5~~ — REMOVED from this ticket.
   Ticket 02 (02-tool-compliance-repeat) owns that run; scheduling it here
   too would double-book 150 fresh conversations against the thread budget.
6. Log every result in `docs/hypotheses.md` §7 rows / §F-sections with
   sample size, dates, `serviceVersion`; promote conclusive findings to
   `docs/m365-copilot-api.md`.

## Acceptance
- Each of the four in-scope probes run on a rested account, one variable per run.
- Results logged in `docs/hypotheses.md` with n per cell.
- Conclusive findings promoted to `docs/m365-copilot-api.md`.

## Evidence
- `frame-dump-out/<ts>/`, `tool-compliance-out/<ts>/` (gitignored);
  hypotheses.md §2 (F5), §F6, §7; api doc on any promotion.

## Risks
- Total cost ≈22 msgs after the step-5 deferral; run the 0-msg and 1-msg probes first.
- n=1 noise: latency/compliance claims only from `--repeat ≥3`; rotate
  variant order across repeats.
- Do not interleave probes or fire concurrent threads; any Disengaged or
  empty-reply spike pauses the batch (cooldown, then resume).
- This batch only measures prompts/output — no execution-gating change; the
  frozen 8H fail-closed verifier and gate order are untouched.
