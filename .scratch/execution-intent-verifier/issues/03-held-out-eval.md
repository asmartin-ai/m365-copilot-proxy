# 03 — Held-out evaluation (32 cases)

**Status:** resolved (2026-08-09 — held-out evaluation run through the merged default-on path: unsafe-FP 0, true selective accuracy 0.969, coverage 1.0, stability 1, 15/16 near-pairs)
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

- [x] Run the held-out cases through the merged verifier path — `run-heldout.mjs`
      (production `getIntentVerifier()` singleton, default-on; `produceToolPath` gate)
- [x] Report unsafe-FP count, true selective accuracy, coverage/stability —
      unsafe-FP **0**, selective accuracy **0.969**, coverage **1.0**, stability **1**,
      pairs **15/16 correct, 1 mixed**
- [x] Log the result in `docs/hypotheses.md` with sample size + evidence — §16 (n=32, one frozen run)
- [x] No corpus/prompt/split changes during evaluation — validate-split PASS before and after; frozen corpus/prompt/model byte-identical

## Result (2026-08-09, single frozen run, no M365)

`run-heldout.mjs` (no args; defaults = frozen contract) against exact laptop
`bonsai-27b-q1` @ `127.0.0.1:1234` (llama.cpp b10321, `--seed 42 -ngl 99 -c 8192`),
32 held-out cases / 16 near-pairs, seed 42, temp 0, max_tokens 2048:

| metric | value |
|---|---|
| unsafe execution FP | **0** (ids: none) |
| true selective accuracy | **0.969** (31/32 covered-correct) |
| coverage | 1.0 (32/32; UNCERTAIN 0, INVALID 0) |
| stability | 1 |
| execute recall / text recall | 0.938 (15/16) / 1.0 (16/16) |
| near-pairs | 15/16 correct, 1 mixed (troub-01: EXECUTE→TEXT; its TEXT member correct) |
| latency | median 24661 ms, p95 35927 ms |
| sample size | n=32 (one run) |
| model identity | bonsai-27b-q1 (exact, echoed) |

The only EXECUTE miss is `execution_intent-129` (troub-01/conditional-narrative
shape) → TEXT; fail-closed arbitration means that miss is safe (no execution).
Artifacts: `experiments/tool-decision/execution-intent/results/heldout-8h.{json,md}`;
heartbeat: `%TEMP%\heldout-status.jsonl`.

**Out of scope:** flipping default-on without 01; unfreezing the corpus.