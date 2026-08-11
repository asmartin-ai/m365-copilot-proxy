# 01 — Review findings + session loose ends for triage

**Status:** resolved (2026-08-10; items 1–3 fixed, item 4 honored)
**Category:** enhancement
**Type:** mixed
**Blocked by:** —

## Context

Two-axis code review (2026-08-10, diff `4f0aa83...HEAD`) of the ponytail
cleanup + auth-doc commits. All review findings were fixed in-session
(`143d136`, `e781c4c`, `0d4c32a`). The items below are the follow-ons the
review or the session surfaced that still need a decision or action.

## Findings

### 1. AGENTS.md "bun test = test:unit" is inaccurate — fix the doc

`AGENTS.md` (Build & test) says `bun test  # = test:unit`. In practice
`bun test` runs Bun's **native** runner, which cannot run this vitest
suite: it reports 20 fails / 11 errors on pristine HEAD (vitest-API
incompatibilities: `vi.hoisted`, `vi.mock` factory args). The canonical
suite is `bun run test:unit` (= `bun run build && bun run vitest run`),
which is green (290 pass / 3 skip). An agent following AGENTS.md will
"verify" with a red herring. Fix: correct the shorthand in AGENTS.md and
point at `bun run test:unit`.

### 2. Polluted evidence files — cleanup decision needed

`~/.config/opencode-m365/throttle-telemetry.ndjson` (61 lines, ALL
unit-test events — convIdHash = sha256("handler-conversation")) and
`~/.config/opencode-m365/session-state.json` (629 fixture sessions:
`app-*`/`handler-*`/`response-*`) were written by pre-fix test runs. New
pollution is prevented (vitest.setup.ts, `143d136`), but the existing
content is junk that can mislead. Neither file holds live evidence. Decide:
reset/quarantine both (safe), or leave and rely on the docs note.

### 3. Ponytail item 6 (CdpClient → playwright) — now actionable

`.scratch/de-overengineering/issues/01-ponytail-cleanup-pass.md` item 6
(defer auth's hand-rolled CdpClient to playwright's
`launchPersistentContext`) was deferred because it needs a session that can
run a real interactive login. The next session plans exactly that (M365
auth + live tests). Queue it as ready-for-agent once auth is confirmed.

### 4. Live-test plan needs thread-budget discipline

Next session: auth into M365 on this PC, then live tests + implementation.
Constraints to honor: ~600 messages per conversation cap; ~12 new
conversations/hour (thread-rate throttle, F13); a fresh login does NOT
clear `oid`-keyed throttling; empty response without a `Disengaged` frame
= throttle, not content filter. Live verification entry point:
`M365_DEBUG=1 bun scripts/proxy-verify.mjs --agent --multiturn`. The
attestation proof-header must be re-verified live (laptop smoke predates
the proof-header change).

## Acceptance

- [x] Each finding dispositioned (fix / wontfix / defer) in a follow-up
      session; decision recorded here.
- [x] (item 2) evidence files either quarantined or explicitly kept with a
      pointer to the docs note.

## Comments

- Source: code-review skill run 2026-08-10 + session wrap-up.
- Item 1 and item 3 are ready-to-execute; item 2 is a one-line decision;
  item 4 is a constraint reminder, not code.
- 2026-08-10 dispositions: item 1 FIXED — AGENTS.md now points at
  `bun run test:unit` and documents the bare-`bun test` trap (native runner,
  not the vitest suite). Item 2 FIXED — `throttle-telemetry.ndjson` +
  `session-state.json` moved to `~/.config/opencode-m365/quarantine-2026-08-10/`
  (reversible; both were unit-test pollution with no live evidence). Item 3
  DONE — auth CdpClient → playwright landed (TDD, seams `extractAuthCode` +
  `waitForAuthCode`, 8 tests); the interactive-login smoke PASSED on the PC
  (visible Chromium, nativeclient code scraped, token acquired, msal-cache
  written 20:11), and the live `proxy-verify --agent --multiturn` tool loop
  passed end to end with the 8H verifier on LM Studio
  (`M365_INTENT_VERIFIER_MODEL=bonsai-27b`). Item 4 HONORED — live runs
  sequential, thread-budget capped, verifier gated on LM Studio.
