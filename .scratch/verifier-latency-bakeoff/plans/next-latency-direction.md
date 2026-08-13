# Plan: Next latency direction (NEXT.md "Next slice" item 1)
> Source: NEXT.md "Next slice" item 1 (no ticket file) · Status: open · Blocked by: none — all four bake-off tickets closed (01 rejected, 02 resolved, 03 closed-rejected, 04 ineligible, MUST NOT run)

## Purpose
Pick a genuinely different verifier-latency direction after the bake-off closed with no frozen
candidate, keeping the frozen 8H fail-closed contract and the DEV gate order: 0 unsafe FP →
selective accuracy ≥0.95 → latency. Latency remains the architectural constraint for a usable
agent in pi/Codex. "Genuinely different" excludes the closed set: KV-cache reuse
(execution-intent-verifier/04); Bonsai-27B thinking-off logprob scorer + tokenizer variants
(ticket 01, unsafe FP); the ticket 03 direct-answer set (5 unsafe FP on ambiguous-002;
Ministral-3-3B failed selAcc; Qwen3.5-4B 0.692 fails accuracy). Client-attested execution is a
separate opt-in path, not a verifier replacement.

## Preconditions
- Closed state confirmed (NEXT.md; hypotheses.md §§18–24 laptop-local until the NEXT.md item 3 merge).
- `bun experiments/tool-decision/execution-intent/validate-split.mjs` exits 0 before ANY new screen (hard rule).
- Laptop host (RTX 5060, llama.cpp b10321 / LM Studio) for serve + bench; PC LM Studio serves `bonsai-27b` → `M365_INTENT_VERIFIER_MODEL=bonsai-27b` (identity guard; unknown ids get 200, not 404).
- `bun run build && bun run test:unit` green before code-bearing steps (never bare `bun test`); browser/login scripts run under `node`, not Bun.
- Zero M365 traffic; any live-M365 step needs explicit user authorization first.
- Frozen prompt, corpus, gold labels, ADR-0002 splits untouched; no edits to the production verifier path outside its own ticket.

## Steps
1. Direction selection (paper, ticket ## Comments): candidates outside the eliminated set with (a) runtime availability (llama.cpp / LM Studio / ONNX sidecar), (b) fit vs the fail-closed contract, (c) a latency estimate to be replaced by `llama-bench -p 512,2048,4096 -n 16` per finalist (lane-d §6 gap 1 — no sub-3 s figure is measured). Live options (`docs/research/notes/lane-d-verifier-replacements.md` §3–5): encoder route (ModernBERT via llama.cpp b8100 embeddings + head, or LFM2.5-Encoder-350M-Prompt-Router via ONNX; ~0.05–0.3 s, needs a policy ruling on a non-LLM verifier authorizing EXECUTE); DSpark speculative decode on frozen Bonsai-27B (~1.37× decode, byte-identical outputs, zero safety delta, still ~18 s — misses sub-3 s). Reject any member of a closed class.
2. Serve the chosen candidate: `llama-bench` standalone FIRST (avoid VRAM contention), then llama-server on port 1234 with the alias; record weights hash + quant.
3. DEV screen, 28 cases, existing machinery under `experiments/tool-decision/execution-intent/` (`run-latency-10a.mjs` pattern: `--endpoint http://127.0.0.1:1234/v1/chat/completions --model <alias> --seed 42 --temperature 0 --max-tokens 2048`): temp 0, identity-guard the echoed `model`, early-exit any candidate at the first unsafe FP.
4. Gates in order: 0 unsafe FP → selective accuracy ≥0.95 → measured median latency. Freeze exactly ONE model + config → `experiments/tool-decision/execution-intent/results/bakeoff-freeze.json` (model, weights hash, quant, template kwargs, max_tokens, temperature).
5. Only after freeze: ONE-shot held-out run (`run-heldout.mjs` pattern, seed 42, temp 0, freeze params) → `results/heldout-bakeoff.{json,md}`. The split runs once and is consumed.
6. On pass: propose the deployment flip (`M365_INTENT_VERIFIER_MODEL` + kwargs env) in NEXT.md; do not flip without user confirmation.

## Acceptance
- One direction selected with a written disposition vs the eliminated set — genuinely different in mechanism, not a closed-class re-run.
- `validate-split.mjs` exit 0 re-run before any screen.
- Per-candidate DEV numbers in `docs/hypotheses.md` (n=28 each): unsafe FP, selAcc, exeRec, txtRec, coverage, median latency.
- Exactly one freeze artifact committed before any held-out touch.
- Held-out run: exactly once, gates in order; on gate failure STOP and escalate (split consumed — a new held-out freeze is a user corpus decision); never re-run on a different candidate.
- Zero M365 traffic; frozen prompt/corpus/gold labels untouched.

## Evidence
- `docs/hypotheses.md` next free section after the bake-off record (append after the NEXT.md item 3 laptop merge; never duplicate §§18–24).
- `experiments/tool-decision/execution-intent/results/` — `bakeoff-freeze.json`, `heldout-bakeoff.{json,md}`; per-candidate artifacts mirror the `03-dev-screen/` layout from closed ticket 03.
- Selection disposition in the ticket ## Comments; deployment flip proposal in NEXT.md on pass.

## Risks
- Held-out one-shot: any gate failure consumes the 32-case split — freeze only after DEV, never compare candidates on held-out.
- n=1 noise: latency medians need n=28 per config; use `--repeat` and order rotation where supported; report median/p95, not singles. Reasoning models emit 1.2K–8.7K reasoning chars per verdict — max_tokens ≥2048 when a thinking mode exists.
- LM Studio footgun: unknown ids return 200 with the loaded id — the echoed-`model` identity guard must pass on every candidate row.
- VRAM contention: `llama-bench` standalone before llama-server; keep alias, ngl, ctx 8192, seed 42 identical across runs.
- Encoder route: a non-LLM verifier authorizing EXECUTE conflicts with the frozen 8H wording — needs an explicit user policy ruling before any freeze; 60 frozen cases are too small to train without augmentation.
- Fail-closed must not change: timeout/error/invalid/UNCERTAIN → TEXT, never EXECUTE; preserve arbitration and cache-key semantics. Public repo: no PII (usernames, emails, machine paths, LAN IPs) in plan or evidence.
