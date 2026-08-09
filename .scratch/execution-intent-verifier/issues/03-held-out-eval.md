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

## Runbook (laptop)

Execute on the laptop with the M365 backend + LM Studio / Bonsai on
`127.0.0.1:1234` (same preconditions as ticket 01 §1).

```sh
cd experiments/tool-decision/execution-intent
bun validate-split.mjs                      # split invariants must pass first
bun run-heldout.mjs \
  --endpoint http://127.0.0.1:1234/v1/chat/completions \
  --model bonsai-27b-q1 --seed 42 --temperature 0 --max-tokens 2048
```

Reads `heldout.json` (32 cases / 16 near-pairs), drives each through the
merged production path (`produceToolPath` + `getIntentVerifier`), writes
`results/heldout-8h.{json,md}`. Report unsafe-FP count, true selective
accuracy, coverage, stability, and pair-level correct/mixed counts; log the
result in `docs/hypotheses.md` with sample size + evidence.

## Acceptance

- [ ] Run the held-out cases through the merged verifier path
- [ ] Report unsafe-FP count, true selective accuracy, coverage/stability
- [ ] Log the result in `docs/hypotheses.md` with sample size + evidence
- [ ] No corpus/prompt/split changes during evaluation

**Out of scope:** flipping default-on without 01; unfreezing the corpus.