# 01 — Live validation of the 8H verifier

**Status:** ready-for-human
**Category:** enhancement
**Type:** task
**Blocked by:** —
**Spec:** ../spec.md

## What to build

Run the verifier against real M365 threads on the laptop (M365 backend +
Bonsai / LM Studio verifier available there) and confirm production parity:
tool flows on EXECUTE verdicts, text on TEXT/UNCERTAIN, cache hits served
byte-identical, no throttle interaction introduced.

## Agent Brief

**Category:** enhancement
**Summary:** Validate the opt-in execution-intent verifier on live M365.

**Current behavior:** The verifier gate is implemented and unit-tested but
never exercised against real M365 traffic.

**Desired behavior:** With `M365_INTENT_VERIFIER=1` and the verifier running
locally, a real proxy session emits EXECUTE-gated tool calls, raw text on
non-EXECUTE verdicts, cache hits within a thread, and no new
throttle/Disengaged interaction.

**Key interfaces:**
- `getIntentVerifier()` env/endpoint gating in
  `packages/proxy-lib/src/intent-verifier.ts`
- Tool-path gate in `tool-path.ts` (non-EXECUTE → raw text)

**Acceptance criteria:**
- [ ] Verifier EXECUTE results in the tool call executing end-to-end
- [ ] Verifier TEXT/UNCERTAIN results in raw text, no execution
- [ ] Repeat requests hit cache (0 ms, identical body) within one thread
- [ ] No new Disengaged/throttle behavior attributable to the verifier
- [ ] Fail-closed path verified live: verifier down → text, not execution

**Out of scope:**
- Flipping the default-on state (ticket 02)
- Held-out evaluation (ticket 03)
- Corpus/prompt changes (frozen)