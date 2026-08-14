# 09 — Custom-instructions probe (user-proposed)

**Status:** ready-for-human
**Type:** research
**Category:** enhancement
**Blocked by:** laptop reconnect + multi-omp session setup (user), then
explicit execution authorization

## Context

User hypothesis: account-level Custom Instructions could make M365's output
format friendlier for tool parsing. Lane F
(`docs/research/notes/lane-f-custom-instructions-lever.md`) confirms:

- Custom Instructions is part of Copilot Memory, stored in the Exchange
  mailbox, retrieved server-side by oid.
- Two independent wire captures show the `add_custom_instructions` optionsSets
  flag on agent-less turns of this exact endpoint.
- The kuchris reference proxy sends the flag on every API turn. **This proxy
  sends none of these flags today.**
- Override plumbing already exists: `M365_EXTRA_OPTIONSSETS` and
  `scripts/_probe-chat.mjs` optionsSets support — probe-ready, zero new code.

## Design (corrected per judge finding 3 — the original 4-thread draft is a PILOT only)

1. **Zero-thread pre-flight:** open the Copilot GUI Custom Instructions panel.
   Greyed-out toggle = tenant disabled it → abort, zero threads spent.
2. **Pilot (≤4 threads):** one flag-on / flag-off pair, rested account,
   generous cooldown. Purpose: does the server accept the flag and does the
   instruction visibly shape responses? Pilot results CANNOT conclude the
   hypothesis — n=1 is noise (AGENTS.md).
3. **Replicated runs:** ≥3 flag-on / flag-off pairs per question, rotated
   order, rested account, one thread at a time. Thread-rate throttle onset =
   that run is INCONCLUSIVE (F24: back-to-back fresh conversations measure
   throttle state, not treatment) — wait it out, then resume.
4. Tone and full-flag-triplet questions get separate replicated runs, never
   bundled.

**Budget cap:** pilot + replicated runs ≤ ~12 threads total, sequential only.

## Acceptance

- [ ] Pre-flight outcome recorded (enabled/disabled) with zero threads spent
- [ ] Pilot logged as pilot (explicit "cannot conclude" marker)
- [ ] Replicated runs logged in `docs/hypotheses.md` with n per arm and
      evidence pointers
- [ ] Conclusive findings promoted to `docs/m365-copilot-api.md`

## Comments

- 2026-08-09: User will reconnect the laptop and create the multi-omp-session
  setup. Standby until then; do not schedule M365 threads before explicit
  execution authorization.
