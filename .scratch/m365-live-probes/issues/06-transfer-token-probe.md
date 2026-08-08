# 06 — ConversationTransferToken migration probe

**Source:** `docs/hypotheses.md` §7 row 🔴; F8
**Blocked by:** laptop (~5 msgs)
**Type:** research
**Category:** enhancement

## Goal

Try to POST `conversationTransferToken` to various Sydney paths to see if a
conversation can be migrated — the 600-msg-cap workaround (a long loop could
harbor in a new thread and keep the context via transfer token).

## Acceptance

- [ ] Script written; candidate paths enumerated and probed
- [ ] Migration success/failure per path recorded
- [ ] F8 verdict updated in `docs/hypotheses.md`