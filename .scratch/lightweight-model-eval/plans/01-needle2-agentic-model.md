# Plan: Evaluate Needle 2 as a hyper-lightweight verifier candidate
> Ticket: .scratch/lightweight-model-eval/issues/01-needle2-agentic-model.md · Status: backlog · Blocked by: binary fetch (cactuscompute.com/needle); a dev-only screen runner; local inference host (CPU-only is the interesting case — 70 MFLOPs/token claim)

## Purpose
Decide whether Cactus Compute Needle 2 (14 MB CQ2-bit, 45 M params,
tool-call-trained) is a viable hyper-lightweight EXECUTE/TEXT/UNCERTAIN
verdict model for the verifier-latency bakeoff: run it on the frozen 28-case
DEV corpus with locally measured decode speed, compare against the recorded
Bonsai/Qwen3.5 baselines, and record a viable/partial/not verdict. A
14 MB verdict model changes bakeoff economics for a usable agent in pi/Codex
(latency is the architectural constraint). Zero M365 traffic.

## Preconditions
- Offline eval: no user authorization required, no M365 calls. NO live probes
  of any kind.
- Standing rules: run `bun experiments/tool-decision/execution-intent/validate-split.mjs`
  before the screen; frozen gate order — 0 unsafe false positives → selective
  accuracy ≥0.95 → latency; held-out MUST NOT run (04-heldout-gate is one-shot
  and ineligible — never load heldout.json); identity-guard the echoed model
  on every response; keep weights/hash manifests out of git (public repo).
- Vendor specs are self-reported and unverified — measure locally before
  believing.

## Steps
1. Fetch the Needle 2 engine/binary from cactuscompute.com/needle; record
   file size + hash. Verify the 14 MB / 28 MB RAM / CQ2-bit claims
   independently (on-disk size; engine-reported memory; quant label).
2. Serve/run locally: engine server mode if OpenAI-compatible (then the
   intent-verifier env contract works: `M365_INTENT_VERIFIER_ENDPOINT`,
   `_MODEL`, `_MAX_TOKENS`, `_TIMEOUT_MS`, optional
   `M365_INTENT_VERIFIER_TEMPLATE_KWARGS`); else drive the engine CLI per
   case. CPU-only first — no GPU needed at this size.
3. Decode-speed bench (local, not vendor numbers): fixed prompt+completion,
   timed, ≥3 reps, order-rotate; report median tok/s and engine-reported
   MFLOPs/token if exposed.
4. Dev screen: 28 cases from `experiments/tool-decision/execution-intent/dev.json`,
   temp 0, seed 42, max_tokens ≥2048. Use a dev-only runner modeled on
   `run-fail-closed-8h.mjs` / `run-latency-10a.mjs` (dev.json only; heldout
   hard-rejected) hitting the verifier contract, or the merged production path
   (`produceToolPath` + `getIntentVerifier`). Early-exit on the first unsafe
   FP — do not burn remaining cases. Count INVALID/UNCERTAIN separately;
   identity-guard the echoed model id.
5. Metrics (frozen README semantics): unsafe_execution_fp, selective_accuracy,
   coverage, execute/text recall, median + p95 latency. Compare against
   recorded baselines (Bonsai-27B, Qwen3.5-4B, qwythos-9B, LFM2.5-2.6B —
   hypotheses §§15–19 + ticket-03 evidence).
6. Optional probe: if the engine exposes its tool-call grammar, parse its
   tool-call output with the existing fenced parser (packages/core fenced.ts +
   parseToolCalls) to test format generalization.
7. Record verdict in `docs/hypotheses.md` (verifier-candidate section, §§19–24
   area): viable / partial / not, with n=28, numbers, and evidence pointer.

## Acceptance
- Binary fetched; size/RAM/quant claims verified locally (recorded, not
  vendor-cited).
- 28-case dev run recorded: unsafe FP, selAcc, exeRec, txtRec, coverage,
  median/p95 latency, n.
- Decode tok/s + MFLOPs measured locally.
- Verdict recorded in `docs/hypotheses.md` with sample size + evidence
  pointer; held-out untouched; zero M365 traffic.

## Evidence
- `experiments/tool-decision/execution-intent/results/` (dev-screen JSON + MD),
  `docs/hypotheses.md`, ticket `## Comments`. Hash manifest kept out of git.

## Risks
- CQ2-bit 45 M model may not clear the gate — the early-exit rule caps the
  cost (0 unsafe FP is the hard gate; selective accuracy ≥0.95 is a high bar).
- No OpenAI-compatible server mode → CLI wrapper work (budget it in step 2).
- Identity-guard footgun if served via LM Studio (echoed-id mismatch).
- Latency/decode variance at n=1 → ≥3 reps + order rotation; report median.
- One-shot held-out: never run heldout.json for this candidate; screen on DEV
  only, freeze one survivor before any held-out consideration (ineligible here).
- Public repo: no PII; keep weights and local hash manifests out of git.
