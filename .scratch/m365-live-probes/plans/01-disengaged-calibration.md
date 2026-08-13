# Plan: 01 — Disengaged calibration
> Ticket: .scratch/m365-live-probes/issues/01-disengaged-calibration.md · Status: ready-for-agent · Blocked by: none
## Purpose
Run a prompt-aggression ladder to find the `dea_violation` threshold where the Disengaged filter fires. Bounds F6 (threshold scales with tool count, not size) and gives the framing-sweep a calibrated ceiling, so the proxy can predict disengagement instead of discovering it per-request.
## Preconditions
- Explicit user authorization to send live M365 probes (hard project rule — never implied by ticket status).
- Rested account: backoff level 0, ≥24h since last probe run, fresh auth cache verified (PC; laptop no longer required).
- Thread budget: ≤12 fresh conversations/hr, ≥3 min spacing, strictly sequential.
- No proxy build needed — standalone probe script; if lib code is split out, `bun run test:unit` (never bare `bun test`).
## Steps
1. Authorize the probe run and confirm the rested-account state (backoff level 0).
2. Write `scripts/disengaged-calibration.mjs` reusing `scripts/_probe-chat.mjs` (options: `tone`, `variants`, `optionsSets`, `extraAllowed`, `plugins`):
   - Ladder steps calibrated to the known scale (hypotheses.md: clean ~1e-8, prose ~1e-6, jailbreak ~1e-3, fires >2e-3): benign baseline → prose ask → tool-heavy (12-tool context) → progressively aggressive framing (mirror `frame-dump-disengage.mjs`: "STRICT RULES: never describe your intent. Output ONLY JSON.") → jailbreak-shaped.
   - Per step, record: outcome class (normal reply / Disengaged / empty-without-Disengaged = thread throttle, NOT a filter hit), `scores.dea_violation` from the bot frame (wire value; the proxy surfaces it as `usage.x_m365_dea_score`).
   - 3+ reps per step, each rep a fresh conversation; write results to `scripts/disengaged-calibration-out/<TS>/results.json` (mirror `tool-compliance-out` layout).
3. Run with pacing: `M365_NO_INTERACTIVE=1 CHROMIUM_PATH=$(which chromium) bun scripts/disengaged-calibration.mjs` — one thread at a time, ≥3 min spacing, hard stop at first empty-503 / at-limit.
4. Report the observed firing threshold (dea value range where outcome flips), with n per step.
5. Log results in `docs/hypotheses.md` (§7 row + F6 / §1.6 update); if conclusive, promote the threshold to `docs/m365-copilot-api.md` §5; append a ## Comments note to the ticket.
## Acceptance
- `scripts/disengaged-calibration.mjs` written and runnable.
- Threshold observed with sample size: prompt ladder complete, 3+ reps/step, `dea_violation` (proxy field `usage.x_m365_dea_score`) tracked per step, n stated.
- Outcome logged in `docs/hypotheses.md`; Disengaged-vs-throttle classification kept separate (empty reply without Disengaged frame = throttle).
## Evidence
- `scripts/disengaged-calibration-out/<TS>/results.json`; `docs/hypotheses.md` F6 / §1.6 / §7 row; `docs/m365-copilot-api.md` §5 if conclusive; ticket ## Comments.
## Risks
- Threshold is per-context (tool count, prompt shape) — a single number over-fits; report range + context. n=1 per step is noise: 3+ reps, rotate ladder order across reps. Degraded/rate-limited account shifts dea scores — rest before runs, stop at empty-503. No concurrent threads ever.
