# 03 — Usage-endpoint hunt v2 (full browser headers)

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** laptop (0 msgs — GETs)
**Source:** `docs/hypotheses.md` §7 row 🔴; F5

## Goal

Same as `usage-endpoint-hunt.mjs` but with full browser headers
(Origin / User-Agent / Accept-Language). F5 is currently low-confidence;
this settles whether a token-usage endpoint exists.

## Acceptance

- [ ] Script written with browser headers (reuse `_probe-chat.mjs` harvest)
- [ ] Every candidate endpoint tested, verdict per endpoint
- [ ] F5 updated in `docs/hypotheses.md`