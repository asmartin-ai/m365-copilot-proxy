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

## Runbook (laptop)

Verbatim-executable procedure for the laptop host. Execute this ticket on the
machine with the M365 backend, not the workstation.

### 1. Preconditions

- LM Studio (or any OpenAI-compatible server) with **`bonsai-27b-q1`** loaded,
  listening on `127.0.0.1:1234`. Identity-guard the echoed `model` field —
  LM Studio silently serves the currently-loaded model for unknown model ids
  (footgun); a mismatch makes the verifier error — safe (TEXT), but
  invalidates this run.
- M365 auth cache present: `~/.config/opencode-m365/msal-cache.json` (fresh
  token; if expired run `bun packages/proxy/bin/m365-login.mjs` interactively
  — do NOT attempt to paste tokens).
- `CHROMIUM_PATH` only if the host's bundled Playwright browser is
  unavailable (auth/WebSocket bootstrap only).

### 2. Environment

From the repo root:

```sh
export M365_INTENT_VERIFIER=1
# Optional overrides (defaults below are the frozen policy values):
# export M365_INTENT_VERIFIER_ENDPOINT=http://127.0.0.1:1234/v1/chat/completions
# export M365_INTENT_VERIFIER_MODEL=bonsai-27b-q1
# export M365_INTENT_VERIFIER_TIMEOUT_MS=${MAX}
export M365_DEBUG=1          # Watch the Disengaged filter
```

### 3. Start the proxy

```sh
bun run build
bun scripts/install-hooks.mjs   # pre-push secret scan (if not already)
bun packages/proxy/bin/m365-proxy.mjs
```

### 4. Drive the session

One long thread — never loop fresh conversations (AGENTS.md throttle F13):

```sh
bun scripts/proxy-verify.mjs --agent --multiturn
```

### 5. Expected-log assertions

From the debug log at `~/.config/opencode-m365/debug.log` (with
`M365_DEBUG=1`). Two log lines carry verifier state — the **structured
record** (lowercase `intent-verifier:` prefix, emitted inside `check()`)
and the **gate log** (title-case `Intent verifier:`, emitted at the gate;
decision only):

- `intent-verifier: model=… policyVersion=8h promptHash=… responseHash=…
  cache=miss decision=EXECUTE latencyMs=… error=null reasoningChars=…
  ts=…` → the tool call executes. Assert the literal `intent-verifier:`
  prefix and its fields (`model`, `policyVersion`, `promptHash`,
  `responseHash`, `cache`, `decision`, `latencyMs`, `error`,
  `reasoningChars`, `ts`).
- The gate then logs `Intent verifier: decision=EXECUTE` (title-case,
  decision only — no record fields).
- `Intent verifier denied execution (TEXT|…)` → raw text returned, no
  execution (this is the 8H arbitration line).
- `Intent verifier authorized execution of N tool call(s)` → EXECUTE path
  completed.
- `cache=hit` (or `cache=shared` on an identical in-flight check) inside
  the `intent-verifier:` record on a repeated tool-shaped turn within the
  same thread.
- `intent-verifier drift: responseHash changed` fires when the planner
  text under the same prefix key changed between requests; it treats the
  cached result as stale and re-verifies. Only logged when that happens —
  its absence on repeat turns is the cache-hit signal.

### 6. Fail-closed drill

1. Stop the LM Studio server (or `killall bonsai`).
2. Send the same tool-shaped turn again.
3. Assert: the proxy returns raw text — the `intent-verifier:` record shows
   `decision=TEXT` with a non-null `error` (`timeout` / `network` /
   `HTTP 500` / `HTTP 503`) — and NO tool executes.
4. Restart the server; the next tool-shaped turn recovers to EXECUTE.

### 7. Throttle watch

- Check for `messageType:"Disengaged"` in the frames. None should be
  attributable to the verifier (its verdict doesn't change the chat payload).
- Rate: one thread, back-to-back turns. If empty-503s appear, it's
  thread-throttle, not the verifier.

### 8. Pass/fail matrix — maps the 5 acceptance criteria

| Acceptance criterion | Checkpoint |
|---|---|
| EXECUTE flows end-to-end | §5 first assertion + `SOLVED` baseline holds |
| TEXT/UNCERTAIN → raw text, no execution | §5 denial line + manual inspect |
| Cache hits byte-identical within thread | §5 `cache=hit` on repeat |
| No Disengaged attributable | §7 absence check |
| Fail-closed when verifier down | §6 |

### 9. Rollback

Unset `M365_INTENT_VERIFIER=1` (and any `M365_INTENT_VERIFIER_*` override) →
legacy tool path, byte-identical to pre-verifier behavior. No other change
required.

### Status transitions on completion

- All green → mark 01 `resolved`; 02/03/04 unblock (02 flip default, 03
  held-out, 04 latency) become the active frontier.