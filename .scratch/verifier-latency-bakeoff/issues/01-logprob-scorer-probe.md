# 01 — Logprob-scorer probe + latency decomposition

**Status:** ready-for-agent
**Type:** research
**Category:** enhancement
**Blocked by:** —

## Context

Cheapest disposition first (judge finding 10): the current endpoint already
exposes `top_logprobs:8` (NEXT.md). Before downloading new models, test
whether the known Bonsai-27B can verdict fast under (a) thinking switched off
server-side and/or (b) a bounded-decode logprob scorer. KV-cache reuse was
already measured-and-rejected (`execution-intent-verifier/04`) — do not retry
it.

## Change

1. **Measure the decomposition** (currently an estimate). On the laptop
   verifier host: `llama-bench -p 1024,2048,4096 -n 16` on `bonsai-27b-q1`,
   plus prompt-eval vs generation timing from llama-server logs for 3–5 real
   dev-corpus cases. Record both numbers.
2. **Logprob scorer variant.** Request with small `max_tokens` (64),
   `logprobs: true`, `top_logprobs: 8`; classify by first-token logprob mass
   over EXECUTE / TEXT / UNCERTAIN. Evaluate on the 28-case dev corpus
   (temp 0, existing bench machinery under `experiments/tool-decision/`).
3. **Two configurations:** thinking-on (today) and thinking-off via server-side
   `--chat-template-kwargs '{"enable_thinking":false}'` (zero code; Lane D
   verified the template switch is valid). Compare verdict latency and corpus
   outcomes.

## Acceptance

- [ ] Decomposition measured and recorded (prefill t/s, decode t/s, per-case
      timings) — the research doc's "~2 s prefill + ~23 s decode" is confirmed,
      corrected, or killed with numbers
- [ ] Logprob scorer evaluated on the 28 dev cases in both configurations
- [ ] Disposition written: pass = 0 unsafe FP on dev AND median verdict < 5 s;
      otherwise reject with numbers and move to ticket 03 candidates
- [ ] Results logged in `docs/hypotheses.md` with sample size (n=28 per config)
      and evidence pointers
- [ ] Zero M365 traffic; zero code changes to the production verifier path

## Comments
- 2026-08-09 (architect): execution started by laptop implementer. Last known state
  before laptop crash: Phase I in progress — llama.cpp b10321 binaries located,
  Bonsai-27B-Q1_0 GGUF located, plan fixed (llama-bench standalone FIRST to avoid
  VRAM contention with llama-server, then verifier host on port 1234 alias
  bonsai-27b-q1 ngl 99 ctx 8192 seed 42, then per-case timings). Merge of main
  (b2ca3f7) landed clean before the crash; build/test validation outcome not yet
  reported. On resume: verify merged tree (bun run build + test:unit, expect
  239 pass / 3 skip), then restart Phase I from the top — no partial results were
  recorded.
