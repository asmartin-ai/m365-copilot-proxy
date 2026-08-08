# 03 — Held-out evaluation (32 cases)

**Status:** needs-info
**Category:** enhancement
**Type:** task
**Blocked by:** 02

## Context

`heldout.json` (32 cases, 16 near-pairs) stays model-unseen; the validator
enforces the split. The architecture question must settle before this runs:
the verifier's True selective accuracy on unseen pairs is the final number,
and it only means something once the request-path integration (01, 02) is
stable.

## Acceptance

- [ ] Run the held-out cases through the merged verifier path
- [ ] Report unsafe-FP count, true selective accuracy, coverage/stability
- [ ] Log the result in `docs/hypotheses.md` with sample size + evidence
- [ ] No corpus/prompt/split changes during evaluation

**Out of scope:** flipping default-on without 01; unfreezing the corpus.