# Plan: Route-only output-boundary steering (no prose rewriting)
> Ticket: .scratch/programmatic-injection/issues/03-output-routing.md · Status: resolved (offline slices 2026-08-11; live baseline capture deferred pending authorization) · Blocked by: none (complements 02; `system_fingerprint` contract defined there)

## Purpose
Harden the shipped fenced ```bash``` shell-routing into a disciplined
output-boundary contract: route/interleave, never rewrite; attribute steered
responses via `system_fingerprint`; guard against Disengagement/dea drift when
injection is active. This is the output side of the ladder — steering the
usable agent in pi/Codex without touching the undocumented backend.

## Preconditions
- Live M365 work needs explicit user authorization (standby-only); strictly
  sequential, one thread at a time, ≤12 fresh conversations/hr, ≥3 min spacing,
  hard stop at first empty-503/at-limit. Empty reply without a Disengaged frame
  = thread throttle, not content filter.
- Baseline-vs-injected comparison needs telemetry: `usage.x_m365_dea_score`
  (already emitted in proxy-lib `usage-builder.ts` from `scores.dea_violation`)
  and Disengaged events (already emitted in proxy-lib `handler.ts` →
  `emitThrottleEvent`).
- 8H fail-closed verifier frozen; execution gating keeps gate order: zero
  unsafe false positives → selective accuracy ≥0.95 → latency. Tool calling
  stays prompt-emulated fenced-shell routing; no native Studio/MCP/Dataverse
  tool-calling (license cost). Verify `bun run build && bun run test:unit`.

## Steps
1. Keep existing fence-routing unchanged (no regression): ```bash``` → shell
   tool via core `fenced.ts` (SHELL_LANGS) → proxy-lib `tool-path.ts`
   (`isProseDocument` guard, `M365_ALLOW_MULTI_TOOL`, mixed-output text strip).
   NEVER rewrite Copilot prose into tool calls — that fabricates intent and
   breaks multi-turn.
2. Emit `system_fingerprint` per the ticket-02 contract: top-level field on
   chat-completion JSON + Responses envelope, sourced from CopilotSession
   steering state (`steered:channel=X` | `unsteered`); mirror as
   `x_m365_system_fingerprint` in usage for streamed chunks.
3. Steering-attribution gate in `tool-path.ts`: when the ladder is active
   (`M365_STEERING` on), route a parsed fence ONLY when the response is
   attributable as steered; unsteered responses degrade to raw text (honest
   degrade). When steering is disabled, preserve legacy routing exactly.
4. Disengagement/dea-drift guard: a telemetry sink keyed by fingerprint bucket
   (baseline vs steered) accumulating Disengaged events + `dea_violation`
   scores; alert when the steered window's Disengaged rate or mean dea score
   exceeds baseline by a threshold. Rationale: F22 — Prompt-Shields is an
   additive shape classifier; injection text that adds override-imperatives
   raises the score. The guard is observational; it must NOT auto-fail-closed
   the ladder.
5. Baseline capture: N turns steering-off (`M365_STEERING=0`) then N steering-
   on, rotated order, rested account, same task mix, via `scripts/_probe-chat.mjs`
   (returns `scores` + `disengaged` per turn) or proxy telemetry; record in
   `docs/hypotheses.md`.
6. Document the fingerprint + drift contract in `docs/m365-copilot-api.md`
   (output-boundary section).
7. `bun run build && bun run test:unit`; live after authorization:
   `M365_DEBUG=1 bun scripts/proxy-verify.mjs --agent --multiturn`.

## Acceptance
- Existing fence-routing tests unchanged and green (no tool-loop regression).
- Steered responses carry `steered:channel=X`; unsteered carry `unsteered`;
  both observable on chat-completion and Responses envelopes.
- Attribution gate: unsteered responses are not routed to tools when the
  ladder is active; legacy routing intact when disabled.
- Drift check emits a baseline-vs-steered delta for Disengaged rate and
  `x_m365_dea_score` with an alert on threshold; result recorded.
- Full suite green; live `proxy-verify.mjs --multiturn` passes.

## Evidence
- `docs/hypotheses.md` — new § output-boundary drift guard (baseline vs
  steered numbers).
- `experiments/output-routing/` — per-run drift deltas (n, rotation order).
- Ticket ## Comments — alert triggers, fingerprint samples.

## Risks
- n=1 noise: replicate runs ≥3, rotate order, rest the account; a threshold
  set from one session is not a conclusion.
- Injection legitimately shifts Prompt-Shields shape (F22) — set the alert
  threshold from baseline first; do not auto-fail on drift.
- Drift guard must read Disengaged frames + dea scores only; empty-503
  (thread throttle) is not a content-filter signal.
