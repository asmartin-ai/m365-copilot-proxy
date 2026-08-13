# Plan: ConversationTransferToken migration probe
> Ticket: .scratch/m365-live-probes/issues/06-transfer-token-probe.md · Status: ready-for-agent · Blocked by: none

## Purpose
Test whether a conversation can be migrated across Sydney sessions/paths via
`conversationTransferToken` — the candidate workaround for the 600-msg-per-
conversation cap (H8.18): a long agent loop could hop to a fresh conversation
and carry context through the token. Token shape (F8):
base64(`{"type":"FullConversation","conversationId":"<uuid>"}`); mechanism
unknown.

## Preconditions
- Explicit user authorization for live M365 probes (standing rule).
- Rested account: fresh MSAL cache (2026-08-10), backoff level 0; confirm
  with one clean turn before probing (F24: back-to-back fresh threads
  self-throttle and poison results).
- Strictly sequential, one thread at a time, ≤12 fresh convs/hr, ≥3 min
  spacing; hard stop at first empty-503/at-limit.

## Steps
1. Write `scripts/transfer-token-probe.mjs` reusing `scripts/_probe-chat.mjs`
   oneTurn plumbing (optionsSets / extraAllowed / plugins / variants / tone
   overrides). Fresh conversation per turn, structured result out, results
   into a timestamped out dir.
2. Enumerate candidate migration surfaces in code:
   - WS handshake URL query params (`conversationTransferToken=`,
     `transferToken=`, `migrationToken=`) on the Chathub connect;
   - payload fields (`conversationTransferToken`) on `/sydney/v1/...`
     request bodies;
   - REST POSTs to the Sydney siblings `usage-endpoint-hunt.mjs` already
     enumerated (expect header-independent 500s; record anyway).
3. Run sequentially:
   a. baseline turn in a fresh conversation — capture a real
      `conversationTransferToken` from the response frames;
   b. migration attempt: new ConversationId + token on each candidate
      surface; plant a unique sentinel in the source conversation's context;
   c. per-path verdict: success (sentinel recalled in the migrated
      conversation), error/500, or silent ignore.
4. Update F8 in `docs/hypotheses.md` (§F8 table + §7 row 06): paths tried, n
   per path, `serviceVersion` captured from frames.

## Acceptance
- `scripts/transfer-token-probe.mjs` written; candidate paths enumerated and
  probed on a rested account.
- Migration success/failure per path recorded with sample size.
- F8 verdict updated (⚫ dead / 🟡 needs another angle / ✅ works).

## Evidence
- `scripts/transfer-token-out/<ts>/` results (gitignored, pattern of
  `frame-dump-out/`); `docs/hypotheses.md` §F8 + §7 row 06; ticket Comments.

## Risks
- Token is opaque/undocumented — server may reject or ignore unknown fields
  silently; a silent ignore is a verdict, not a retry loop.
- n=1 noise: any "success" needs a second fresh-conversation confirm with a
  unique sentinel; rotate attempt order across runs.
- A successful migration mutates per-conversation state — one attempt per
  conversation, no hammering.
- Migration success only proves the mechanism; the 600-cap itself stays
  unverified (H8.18 requires a ~590-msg conversation).
- This probe only measures the channel — no execution-gating change; the
  frozen 8H fail-closed verifier and gate order are untouched.
