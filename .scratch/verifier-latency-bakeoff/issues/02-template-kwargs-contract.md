# 02 — Forward `chat_template_kwargs` from intent-verifier.ts

**Status:** resolved
**Type:** code
**Type:** code
**Category:** enhancement
**Blocked by:** —

## Context

Judge finding 1: thinking-off candidates cannot be selected through the
documented env contract. `packages/proxy-lib/src/intent-verifier.ts` reads only
`M365_INTENT_VERIFIER_ENDPOINT` / `_MODEL` / max-token / timeout / backoff env
vars, and its request body sends no `chat_template_kwargs`. Every
thinking-mode candidate in ticket 03 depends on this.

## Target

`packages/proxy-lib/src/intent-verifier.ts` — env parsing block and the chat
completion request construction.

## Change

1. New optional env var `M365_INTENT_VERIFIER_TEMPLATE_KWARGS` holding a JSON
   object (e.g. `{"enable_thinking":false}`).
2. When set, include `chat_template_kwargs` in the request body; when unset,
   the request is byte-identical to today's.
3. Invalid JSON → throw at verifier construction (loud config error; never
   silently ignored, never fail-closed-as-UNCERTAIN).
4. Cache semantics: no change needed — the cache key already includes
   `responseHash`, and kwargs change the response, not the key.

## Acceptance

- [ ] `bun run test:unit` green at baseline 205 pass + new tests covering:
      kwargs forwarded when set, absent when unset, invalid JSON throws at
      construction
- [ ] `tsc --noEmit -p packages/proxy-lib/tsconfig.json` clean
- [ ] Default (unset) request body unchanged (asserted in a test)
- [ ] Zero M365 traffic; no other behaviour change

## Comments
- 2026-08-09 (PC implementer): resolved. `M365_INTENT_VERIFIER_TEMPLATE_KWARGS`
  added to `intent-verifier.ts` (JSON object → `chat_template_kwargs` in the
  request body; invalid JSON / non-object throws at construction; unset leaves
  the request byte-identical). 4 new unit tests (forwarded / omitted / invalid
  JSON throws / non-object throws). Suite 239 pass / 3 skip, tsc clean.
