# Execution-Intent Verifier (8H fail-closed)

**Status:** Live — implemented, opt-in, pending live validation
**Spec source:** `experiments/tool-decision/execution-intent/integration-plan-10a.md`
**Decision record:** `docs/adr/ADR-0002-EXECUTION-INTENT-VERIFIER.md`

## What the feature is

A local verifier decides `EXECUTE` | `TEXT` | `UNCERTAIN` for every tool
decision the M365 proxy wants to run (see the README contract in
`experiments/tool-decision/execution-intent/`). Only a literal verifier
`EXECUTE` sentence executes a tool call; everything else resolves to the
model's raw text.

Frozen constraints (do not drift):

- Corpus, splits, and the verifier prompt (`prompts/p4-minimal.txt`) are
  frozen; `INTENT_VERIFIER_PROMPT` stays byte-identical to it.
- Verifier NEVER validates its own held-out set (`heldout.json`, 32 cases,
  16 near-pairs).
- No live M365 or Bonsai in tests — only the verifier EXECUTE authorizes
  execution (gate tested with fakes across tool-path tests).

## Current state (2026-08-08)

- Implemented: `packages/proxy-lib/src/intent-verifier.ts`
  (`getIntentVerifier()` / `resetIntentVerifier()`), wired in `tool-path.ts`,
  injected from `handler.ts`.
- Opt-in: `M365_INTENT_VERIFIER=1` (or endpoint override); default OFF keeps
  existing deployments byte-identical.
- 10A latency readiness (dev-only, Bonsai km42 EM): cold median 24.6 s / p95
  42.1 s; cache-hit 0 ms (byte-identical); single-flight dedup verified;
  fail-closed verified on timeout/error/invalid/UNCERTAIN.
- Test baseline: 224 pass / 3 live-gated skip; `tsc` clean;
  `validate-split.mjs` green (28 dev / 32 held-out).

## Ticket map

| # | Ticket | Status |
|---|--------|--------|
| 01 | Live validation (laptop, real M365) | blocked — needs laptop |
| 02 | Flip default-on after live parity | blocked by 01 + separate approval |
| 03 | Held-out evaluation (32 cases) | unauthorized |
| 04 | Latency engineering (the open alternative) | ready when direction chosen |