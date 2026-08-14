# 02 — Fallback-lane decision gate (telemetry readout)

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** ≥1 week of real use from the 2026-08-10 quarantine OR ≥3 throttle episodes (whichever first)
**Plan:** `../plans/02-decision-gate.md`

## Goal

Close the spec's decision gate. Verify the passive throttle telemetry log is
real-use only (zero unit-test pollution), collect the window, then apply the
spec rule: frequent long lulls → open the Option-2 local fallback-lane build
effort; rare/short lulls → close Option 2 as `wontfix` with the numbers.
Ticket 01 shipped the implementation; this ticket is the readout.

## Acceptance

- [ ] Live log shows ≥1 week of real use or ≥3 episodes, zero test-pollution lines
- [ ] Decision recorded per the spec rule in `docs/hypotheses.md` §17 + this ticket's comments
- [ ] Zero M365 traffic attributable to the telemetry path

## Comments

- 2026-08-11: Ticket created to house the decision-gate plan written against
  resolved ticket 01 (keeps the plans/ ↔ issues/ mirror truthful).
