# Plan: Injection-channel ladder controller (canary-verified, self-healing)
> Ticket: .scratch/programmatic-injection/issues/02-injection-ladder.md · Status: resolved (offline slices 2026-08-11; live canary/latch deferred pending authorization) · Blocked by: none (ticket 01 resolved — memory channel dropped; channel 0 proven 2026-08-11)

## Purpose
A controller that reliably injects proxy steering into the M365 turn, choosing
channels and verifying each actually landed. Primary channel is the
custom-instructions textarea (proven, no-approval, durable, server-persisted).
This is the flagged next work toward the usable agent in pi/Codex.

## Preconditions
- **Live M365 probes need explicit user authorization** (standby-only until granted). Canary/latch turns consume live threads: strictly sequential, one thread at a time, ≤12 fresh conversations/hr, ≥3 min spacing, hard stop at first empty-503/at-limit. Empty reply WITHOUT a Disengaged frame = thread throttle, not content filter.
- Browser-driving runs under `node` on the PC (playwright connectOverCDP times out under Bun). Persistent profile `browser-profile-cdp` stays logged in; `scripts/_profile-cdp.mjs` (CDP 9222) holds it open.
- 8H fail-closed verifier frozen; execution gating keeps gate order: zero unsafe false positives → selective accuracy ≥0.95 → latency. LM Studio `bonsai-27b` + `M365_INTENT_VERIFIER_MODEL=bonsai-27b` for live verify.
- Memory-plugin/Graph/Mail channel dead on Basic (ticket 01 E-O2). Tool calling stays prompt-emulated fenced-shell routing; native Studio/MCP/Dataverse tool-calling out of scope (license cost).
- Verify with `bun run build && bun run test:unit` (never bare `bun test`).

## Steps
1. New helper `scripts/set-custom-instruction.mjs` (node): connect CDP 9222 → Chat → Settings → Personalization → Custom instructions → Edit instructions. Write the textarea via the **native value-setter + input/change events** (Runtime.evaluate on the React-controlled field); `tab.fill('')` and bare `new Event('input')` leave Save disabled — silent no-op. Click Save instructions; verify by re-reading the textarea value. Payload via CLI arg or JSON file; no secrets in the script.
2. Steering state module `packages/core/src/steering.ts`: `setCustomInstruction(payload)` invokes the helper out-of-process (child_process, node binary, short timeout); `get/setSteeringState()` over `~/.config/opencode-m365/steering.json` ({channel, payload, breakerState, lastVerifiedAt}). **Rehydration sled:** on session open replay last-good config instead of re-driving the browser.
3. Mapping-canary `verifyChannel()`: each write plants a random codeword→reply mapping that exists ONLY inside the injected text; a probe turn asks for the codeword without showing the reply — recall = landed. Use `scripts/_probe-chat.mjs` options (optionsSets/extraAllowed/plugins/variants/tone/agent; agentId null for plain chat). Use LEAN non-directive plants — directive-heavy plants confounded ticket 01's first run.
4. Ladder state machine in `packages/core/src/session.ts` (owns sendChat/optionsSets, ~line 531): on session open use the proven custom-instructions textarea channel only. The optionsSets gate flag, prompt preamble, and other candidates remain deferred until each has its own writer and mapping-canary path; they MUST NOT act as fallback channels. Pass latches the textarea channel; failure trips its circuit breaker after N.
5. Honest degrade: an open breaker → stamp `system_fingerprint: 'unsteered'`; never silently ship uninjected output. Values: `unsteered` | `steered:channel=textarea`. First-class OpenAI-compatible field: top-level `system_fingerprint` on chat-completion JSON and the Responses envelope, sourced from CopilotSession steering state; mirror as `x_m365_system_fingerprint` in usage for streamed chunks (usage-builder pattern).
6. No regression: fenced shell-routing (core `fenced.ts` + proxy-lib `tool-path.ts`) and 8H verifier (ADR-0002) unchanged. Live end-to-end after authorization: `M365_DEBUG=1 bun scripts/proxy-verify.mjs --agent --multiturn`.
7. `bun run build && bun run test:unit` — full suite green, no new screens.

## Acceptance
- Helper writes + Save succeeds (verified by re-read); an authorized agent-less turn then recalls a fresh nonce plant (mapping-canary pass).
- Ladder state JSON + rehydration sled replay last-good channel on reconnect with no browser write.
- Breaker opens after N failures; `system_fingerprint: 'unsteered'` emitted when all open; `steered:channel=textarea` after a latched pass.
- Unit tests cover state transitions; existing fence-routing + verifier tests untouched and green; full suite green.

## Evidence
- `docs/hypotheses.md` — new § on the injection ladder (channel-0 canary runs).
- `experiments/injection-ladder/` — canary results per run (nonce, verdict, n).
- `~/.config/opencode-m365/steering.json` — redacted state sample in the ticket ## Comments; per-run update logged there.

## Risks
- Thread budget: every canary turn is a live thread — batch, space ≥3 min, hard stop at first empty-503/at-limit.
- n=1 noise (AGENTS.md): replicate ≥3 with rotated order + rested account. H8.15: 8,000-char silent truncation — keep payload lean, verify by canary, not length.
- F22 additive Prompt-Shields shape: override-imperative steering text raises Disengaged/dea — keep plants lean (feeds ticket 03 drift guard).
- Browser lock: one launchPersistentContext per profile dir at a time; second launch on the same dir = "existing browser session" lock.
