# 01 — Disengaged calibration

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
**Source:** `docs/hypotheses.md` §7 row 🔴; F6

## Goal

Progressively more aggressive prompts to find the `dea_violation` threshold
where `Disengaged` fires. Bounds F6 ("Disengaged threshold scales with tool
count, not size") and gives the framing-sweep a calibrated ceiling.

## Acceptance

- [ ] Script `scripts/disengaged-calibration.mjs` written (reuse
      `_probe-chat.mjs`)
- [ ] Threshold observed with sample size (prompt ladder, 3+ reps/step)
- [ ] `usage.x_m365_dea_score` tracked per step
- [ ] Result logged in `docs/hypotheses.md`; conclusion promoted to
      `docs/m365-copilot-api.md` §5 if conclusive