# 04 — Latency engineering for the verifier

**Status:** resolved (2026-08-09 — no offline latency win; candidates dispositioned in
Acceptance: cache-reuse measured/rejected, pipelining impossible, smaller/non-LLM violate
frozen policy or need held-out/live data, bounded concurrency needs unavailable hardware
or live M365)
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

- [x] Measure each candidate against the 10A baseline (cold median / p95,
      cache-hit, single-flight) — **DISPOSITION** (measured + source review):
      - **cache-reuse** (`--cache-reuse 256`): **measured and rejected** — no
        latency win (med 24721→29524 ms, p95 50717→62545 ms); no ≥256-token
        reusable prefix in this workload.
      - **pipelining**: **impossible** — the gate input is the full planner
        text (`tool-path.ts` `check(fullText)`), which exists only after M365
        completes its turn; nothing exists to pre-verify.
      - **non-LLM / smaller verifier**: violates the frozen 8H policy (Bonsai +
        p4-minimal pinned) or needs held-out (ticket 03) / live M365 data.
      - **bounded concurrency > cap-1**: needs unavailable hardware (one local
        GPU serves one request at a time) or live M365 throttle-interaction data.
- [x] Keep fail-closed arbitration intact (no EXECUTE without verifier) —
      measured in both runs: timeout/error → TEXT, never EXECUTE; 8H parity 28/28.
- [x] No held-out/raw prompt drift; corpus frozen — frozen dev corpus +
      p4-minimal used; raw results preserved byte-for-byte.
- [x] Rec in `docs/hypotheses.md` with evidence + sample size — §15 (n=28/run,
      evidence + falsification).

## Measurement — --cache-reuse 256 vs baseline (2026-08-09, offline, no M365)

Prerequisite verified from the laptop-local binary
(`/path/to/llama.cpp/b10321/llama-server.exe --help`): `--cache-prompt`
defaults **enabled** and `--cache-reuse N` ("min chunk size to attempt reusing
from the cache via KV shifting", default 0) requires it — matching official
llama.cpp server docs. Both runs used `run-latency-10a.mjs` (frozen corpus =
28 dev cases, frozen p4-minimal prompt, `bonsai-27b-q1`, temp 0, seed 42,
max_tokens 2048, timeout 120 s); the ONLY change was the server flag.

| Metric | Baseline | + `--cache-reuse 256` |
|---|---|---|
| cold median | **24721 ms** | 29524 ms |
| cold p95 | **50717 ms** | 62545 ms |
| 8H parity | 28/28 (true) | 28/28 (true) |
| unsafe FP | 0 | 0 |
| sample size (cold) | 28 | 28 |
| cache-hit (phase B) | hit, 0 ms, byte-identical | hit, 1 ms, byte-identical |
| single-flight (phase C) | [miss, shared], 1 verifier call | [miss, shared], 1 verifier call |
| single-flight savings | 13983 ms | 15845 ms |
| cache invalidation (phase E) | v1 hit / v2 miss ✓ | v1 hit / v2 miss ✓ |
| fail-closed (timeout/error) | TEXT, never EXECUTE | TEXT, never EXECUTE |
| model identity (echoed) | bonsai-27b-q1 | bonsai-27b-q1 |

**Result: `--cache-reuse 256` did NOT reduce latency — med +19%, p95 +23%**
(24721→29524, 50717→62545). Plausible cause recorded, not assumed: this
workload has no KV-reusable prefix ≥ the 256-token chunk minimum (shared system
prompt ≈ 50 tokens; the 28 case texts are distinct), so KV-shifting reuse had
nothing to trigger on. Single run per config — descriptive, not statistically
robust; per-case variance is large (e.g. execution_intent-011 42.2 s vs 62.5 s).

Raw results: `experiments/tool-decision/execution-intent/results/latency-10a.baseline.{json,md}`,
`latency-10a.cache-reuse-256.{json,md}`; heartbeat/status log:
`%TEMP%\latency-10a-status.jsonl`.