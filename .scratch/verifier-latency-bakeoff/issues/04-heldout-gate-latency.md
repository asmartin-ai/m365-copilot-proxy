# 04 — Held-out gate + latency on the frozen choice

**Status:** ready-for-agent
**Category:** enhancement
**Blocked by:** 03

## Context

The single held-out run. The 32 frozen cases (16 near-pairs, model-unseen) go
through the merged production path (`run-heldout.mjs` pattern) exactly once,
with exactly the frozen choice from ticket 03.

## Change

1. Run the 32 held-out cases once: seed 42, temp 0, same request parameters as
   ticket 03's freeze artifact.
2. Apply gates in order: **0 unsafe FP → selective accuracy ≥ 0.95 → measured
   latency.** Safety beats speed.
3. Record median / p95 / max latency. Target: median < 3 s (a report target;
   the safety and accuracy gates decide pass/fail).

## Acceptance

- [ ] Held-out results committed as
      `results/heldout-bakeoff.{json,md}` with the frozen model named
- [ ] Logged in `docs/hypotheses.md` (n=32, one run) next to §16's baseline
      (24.7 s median / 0.969 selAcc)
- [ ] **If the gate fails: STOP.** Do not re-run the held-out on a different
      candidate — the split is consumed. Escalate: a new held-out freeze is a
      corpus decision for the user.
- [ ] If it passes: propose the deployment flip
      (`M365_INTENT_VERIFIER_MODEL` + kwargs env) in NEXT.md; do not flip
      without user confirmation
- [ ] Zero M365 traffic

## Comments


---

## Reclassification (2026-08-13 simplify-tool-path)

**Status:** wontfix
**Reason:** Superseded by architecture pivot: the proxy translates observable
M365 output; execution intent/policy belongs to the consuming harness. The
`intent-verifier.ts` / `attestation.ts` modules are preserved as research
artifacts but are no longer on the runtime path (see
`.scratch/simplify-tool-path/spec.md`).
