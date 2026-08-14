# 01 — Delete behavioral policy from produceToolPath

**What to build:** In `packages/proxy-lib/src/tool-path.ts`, remove the four behavioral-correctness mechanisms that make the proxy act as an agent instead of a translator, leaving only deterministic translation + transport resilience.

Remove:
- Read-only fallback inference (`readOnlyFallbackToolCall`) and its import.
- The confab / hallucinated-completion / remote-artifact forced-retry loop (regex-driven second M365 turns).
- The post-retry 502 "file mutation without tool call" semantic detector.
- The 8H intent-verifier gate and the client-attestation gate (and their type imports from `intent-verifier.ts` / `attestation.ts`).
- The force-prompt imports (`CONFAB_FORCE_PROMPT`, `HALLUCINATION_FORCE_PROMPT`, `REMOTE_ARTIFACT_FORCE_PROMPT`) once the loop is gone.

Keep (unchanged):
- `parseToolCalls` fence → tool_calls.
- `isProseDocument` document guard.
- One-call-per-turn (`M365_ALLOW_MULTI_TOOL`).
- Steering-attribution gate (`M365_STEERING`).
- `reply` → text conversion.
- `markSent`, `registerToolCalls`.

**Blocked by:** None — can start immediately.

**Status:** done

- [x] `tool-path.ts` no longer imports `IntentVerifier`, `AttestationGate`, `AttestationClient`, the classifier fns, or `readOnlyFallbackToolCall`.
- [x] No local-model / HMAC / classification logic remains in `produceToolPath()`.
- [x] `bun run build` succeeds for proxy-lib.
- [x] `bun run test:unit` — existing translation tests pass (341 passing; 3 skipped = intent-verifier).
- [x] `runTurn` (the M365 boundary) is called exactly once per input (asserted in tests).
- [x] No new flag/mode/config introduced.

**Scope note:** clean cutover also removed the now-dead attestation opts from the
runtime path — `handler.ts` (`executionGate`/`attestationClient`/`attestationProof`
opts dropped), `responses.ts` (`buildResponse`/`handleResponse` opts), and the two
call sites in `index.ts`. The standalone `attestation.ts` module, its unit test
(`attestation.test.ts`), the `handleAttestationRequest` HTTP endpoint, and the
public re-exports in `index.ts` are left in place as research artifacts (not on
the runtime path) per GPT-4o: "delete the integration tests, preserve the module."
