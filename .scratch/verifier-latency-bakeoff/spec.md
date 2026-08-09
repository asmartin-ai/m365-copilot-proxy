# Verifier Latency Bake-off

**Status:** ready — every ticket is offline (zero M365 traffic); runs on the
laptop verifier host (RTX 5060, llama.cpp / LM Studio)
**Source:** `docs/research/2026-08-09-local-models-8gb-blackwell.md` Option 1 ·
`docs/research/notes/judge-review.md` · `docs/research/notes/lane-d-verifier-replacements.md`

## What this effort is

The execution-intent verifier is safe but slow (24.7 s median, 35.9 s p95 —
measured, `execution-intent-verifier/03`). NEXT.md names latency as the
remaining architectural constraint. This effort swaps the verifier for a
direct-answer model that passes the same frozen corpus gates, without touching
the fail-closed invariant, the frozen prompt, or the ADR-0002 split rules.

Ticket 04 of `execution-intent-verifier` resolved "no offline latency win" for
its four candidates; this effort opens a different door those candidates never
covered: direct-answer models and thinking-off configurations, plus a
logprob-scorer probe on the existing model.

## Hard rules (ADR-0002 + adversarial review)

- **Screen on the 28-case DEV corpus only.** The 32 held-out cases run **once**,
  on **one frozen** model/configuration. Never compare candidates on the
  held-out split.
- **Gate order:** 0 unsafe false positives → selective accuracy ≥ 0.95 →
  measured latency. Safety beats speed, always.
- Frozen prompt, corpus files, and gold labels stay untouched.
- **Zero M365 traffic** in every ticket of this effort.
- The latency decomposition (prefill vs thinking-decode) is an **estimate**
  until ticket 01 measures it. Do not cite it as cause before then.
- LM Studio silently serves the loaded model for unknown model ids — every run
  must identity-guard the echoed `model` field (NEXT.md footgun).

## Thinking-off is a contract problem

`packages/proxy-lib/src/intent-verifier.ts` reads only endpoint / model /
max-tokens / timeout / backoff env vars and sends **no**
`chat_template_kwargs` (judge finding 1). Thinking-mode candidates (Bonsai,
Gemma 4, Nemotron) need either server-side template kwargs or the contract
change in ticket 02. Candidates without a thinking mode
(Ministral-3-3B-Instruct) are true env-var swaps today.

## Ticket map

| # | Ticket | Blocked by | M365 traffic |
|---|--------|------------|--------------|
| 01 | Logprob-scorer probe + latency decomposition | — | 0 |
| 02 | `chat_template_kwargs` contract in intent-verifier.ts | — | 0 |
| 03 | Dev-corpus screen, freeze ONE candidate | 01, 02 | 0 |
| 04 | Held-out gate + latency on the frozen choice | 03 | 0 |
