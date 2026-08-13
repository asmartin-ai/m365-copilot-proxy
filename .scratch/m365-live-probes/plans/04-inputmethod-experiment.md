# Plan: 04 — InputMethod / experienceType flip
> Ticket: .scratch/m365-live-probes/issues/04-inputmethod-experiment.md · Status: ready-for-agent · Blocked by: none
## Purpose
Flip `inputMethod` and `experienceType` enum values and watch `dea_violation` / Disengaged outcomes. Tests H1.7/H1.8: whether an input-mode enum can bypass a "chat assistant" classifier or shift routing — a cheap lever for the usable-agent goal if any value lowers the dea surface.
## Preconditions
- Explicit user authorization for live M365 probes; rested account (backoff 0); ~5–10 msgs total — cheap, single session OK.
- Thread budget: sequential, ≥3 min spacing, ≤12/hr.
- Wire envelope defaults (from `scripts/_probe-chat.mjs`): `inputMethod: "Keyboard"`, `experienceType: "Default"` — these are the baselines to flip against.
## Steps
1. Authorize; confirm rested account.
2. Pre-scan: grep the `studio-dig.mjs` capture for the `experienceType` / `inputMethod` values the real UI actually sends (§1.8 note) — only probe values that appear plausible; unknown enum values may be rejected or behave as dead cells.
3. Write `scripts/inputmethod-experiment.mjs` reusing `_probe-chat.mjs` (extend its message envelope; add `--input-method` / `--experience-type` overrides or equivalent):
   - Axis 1: `inputMethod` ∈ {Keyboard (baseline), Voice, Agent} at `experienceType: "Default"`.
   - Axis 2: `experienceType` ∈ {Default (baseline), Agent, BizChatAgent, Programmatic} at `inputMethod: "Keyboard"`.
   - One combined cell (Agent / Agent) if both axes show signal.
   - Same probe prompt across cells (single-turn, fixed text); per cell record outcome class (normal / Disengaged / empty-without-Disengaged = throttle) + `scores.dea_violation`.
4. Run with pacing: `M365_NO_INTERACTIVE=1 CHROMIUM_PATH=$(which chromium) bun scripts/inputmethod-experiment.mjs`; hard stop at first empty-503/at-limit.
5. Log per-value results in `docs/hypotheses.md` §1.7 and §1.8 (update both rows); append ## Comments to the ticket.
## Acceptance
- Script written; every enum value on both axes probed with a fixed prompt.
- `dea_violation` / Disengaged differences recorded per value (with the baseline in the same run — never compared across sessions).
- Data logged in `docs/hypotheses.md` §1.7/§1.8.
## Evidence
- `scripts/inputmethod-experiment-out/<TS>/results.json` (or sibling out-dir per convention); `docs/hypotheses.md` §1.7/§1.8 rows; ticket ## Comments.
## Risks
- n=1 per enum cell: any winner needs a `--repeat 3` confirmation run before it's load-bearing. Enum names are guesses ("Agent"?, "Programmatic"?) — a rejection/empty reply may mean an invalid value, not a filter change; classify accordingly. dea values are stochastic per account state — keep the baseline cell in the same run, order rotated.
