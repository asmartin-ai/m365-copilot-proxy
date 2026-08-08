# 04 — Latency engineering for the verifier

**Status:** ready-for-agent
**Category:** enhancement
**Type:** task
**Blocked by:** 01

## Context
10A showed the remaining constraint is **latency, not safety**: cold Bonsai
verdict ~24.6 s median / 42.1 s p95 is too slow for an unqualified
request-path gate. Caching + single-flight are already in (cache-hit 0 ms).
Options to squeeze further:

- response-scoped prompt caching (server-side, with DRIFT guard)
- pipelined verifier (pre-verify the next decision while the current one
  runs)
- a non-LLM / smaller verifier for the safe subspace (deterministic
  classifier on the frozen corpus is the calibration basis)
- bounded concurrency above the cap-1 (after live validation – throtttle
  interaction is unknown)

## Acceptance

- [ ] Measure each candidate against the 10A baseline (cold median / p95,
      cache-hit, single-flight)
- [ ] Keep fail-closed arbitration intact (no EXECUTE without verifier)
- [ ] No held-out/raw prompt drift; corpus frozen
- [ ] Rec in `docs/hypotheses.md` with evidence + sample size