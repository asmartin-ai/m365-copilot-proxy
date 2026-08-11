# 04 — InputMethod / experienceType flip

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
**Source:** `docs/hypotheses.md` §7 row 🔴; §1.7, §1.8

## Goal

Flip `inputMethod` (`Keyboard` / `Voice` / `Agent`?) and `experienceType`
enums; watch `dea_violation` for changes. Tests whether input-mode enums
shift the Disengaged/dea surface.

## Acceptance

- [ ] Script written; each enum value probed
- [ ] `dea_violation` / Disengaged differences recorded per value
- [ ] Data logged in `docs/hypotheses.md` §1.7/§1.8