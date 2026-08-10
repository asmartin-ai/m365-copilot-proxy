# 01 — Client-attested execution gate

**Status:** resolved (2026-08-09)
**Category:** enhancement
**Type:** task
**Blocked by:** —
**Spec:** ../spec.md

## What to build

Add the opt-in, client-attested execution path from the feature spec. It must
share the current parsed tool-call path and preserve the 8H verifier behavior
for all requests that do not meet every opt-in condition.

## Required code surface

- Carry attestation selection headers from both Nitro routes through the shared
  handler options:
  - `packages/proxy/routes/v1/chat/completions.post.ts`
  - `packages/proxy/routes/v1/responses.post.ts`
  - `packages/proxy-lib/src/handler.ts`
  - `packages/proxy-lib/src/responses.ts`
- Add a bounded, in-memory attestation candidate registry. It owns candidate
  ids, HMAC verification, nonce replay prevention, and state transitions; it
  must not expand `SessionPool` beyond conversation routing.
- Add the loopback-only `POST /v1/attestations` Nitro route. The handler must
  receive the real peer address from Nitro and ignore forwarded-address headers.
  Do not expose this control endpoint through `createApp()`, because a Fetch
  `Request` has no trustworthy peer address.
- Wire the attested path at the existing `produceToolPath()` gate. It replaces
  the 8H check only for a valid explicit attestation selection. All other
  requests keep the current verifier dependency and behavior unchanged.
- Before the shared handler accepts a Chat Completions tool result or a
  Responses `function_call_output`, require its candidate to be `AUTHORIZED`
  and atomically move it to `RESULT_ACCEPTED`.
- Keep tool-call ids stable through Chat Completions and Responses. Responses
  uses `function_call.call_id`, not its `fc_…` item id.

## Acceptance criteria

- [x] Without complete opt-in config and headers, tool-path behavior is
  byte-for-byte equivalent to the existing 8H verifier path.
- [x] With valid opt-in selection, one parsed `bash` tool call creates one
  `PENDING` candidate keyed by its emitted `call_…` id and SHA-256 of the exact
  emitted command. The 8H verifier is not called for that selected request.
- [x] `POST /v1/attestations` allows exactly one request with valid loopback
  origin, HMAC, timestamp, unused nonce, matching client, matching call id, and
  matching digest.
- [x] Missing, invalid, expired, replayed, non-loopback, or mismatched requests
  deny and leave the shell tool blocked. No id-less or command-only fallback
  exists.
- [x] `allow` consumes the attestation, so a replay denies. A later tool result
  succeeds exactly once only from `AUTHORIZED` state and moves the candidate to
  `RESULT_ACCEPTED`; no result can restore or re-authorize it.
- [x] Chat Completions and Responses expose the same authorization id to the
  client (`tool_calls[].id` and `function_call.call_id`).
- [x] Tests cover the new registry, control route, request selection, legacy
  path, both response wires, and all failure cases without M365 or a real local
  model.

## Out of scope

- Harness adapters and their user interface/configuration.
- M365 live calls, corpus changes, or held-out evaluation.
- Direct proxy shell execution.
- Any weakening of DCG or other harness policy.

## Validation

Run focused Vitest files that cover the new registry/route/tool path, then:

```sh
bun run build
bun test
```

The smoke path is a local mocked client request: obtain one attested candidate,
POST a signed approval once, confirm the second identical POST denies, and
confirm a non-opt-in request still takes the injected 8H verifier path.

## Comments

- Resolved 2026-08-09: `bun run test` passed (248 passed, 3 live-gated skipped).
  Focused attestation coverage passed: 42 tests across registry, tool path,
  Chat Completions handler, and Responses wire. No M365 request was made.
