# Plan: Custom-instructions probe (user-proposed)
> Ticket: .scratch/m365-live-probes/issues/09-custom-instructions-probe.md · Status: ready-for-human · Blocked by: laptop + multi-omp session setup (user), then explicit execution authorization

## Purpose
Test whether a saved account-level Custom Instruction — a persistent,
server-side format lever — makes the agent-less BizChat path emit fenced
```bash tool blocks, replacing per-request framing and lowering
`dea_violation` (lane F H-CI). NEXT.md carry-over (2026-08-11): the channel
is PROVEN — saved instructions apply to raw agent-less WS turns by default
and the `add_custom_instructions` flag is NOT required. This probe therefore
measures the lever's effect (fence emission, dea, Disengaged) and whether
the flag is any gate — not whether the channel exists.

## Preconditions
- User: reconnect laptop + multi-omp session; then EXPLICIT execution
  authorization before any M365 thread is scheduled (ticket comment; NEXT.md
  keeps probes standby-only until then).
- Rested account, backoff level 0; auth via MSAL cache + persistent profile.
- Rig per `docs/agents/m365-ui-investigation.md`; ideation cross-check
  `docs/research/2026-08-11-m365-injection-ideation.md`.
- CDP write technique (NEXT.md, proven): Settings → Personalization textarea
  is React — set value via the NATIVE value-setter, then dispatch `input` +
  `change` events. `tab.fill('')` alone or `new Event('input')` alone leave
  Save disabled (silent no-op; first save AND first clear failed this way).
- Strictly sequential, one thread at a time; ≤12 fresh convs/hr, ≥3 min
  spacing; hard stop at first empty-503/at-limit.

## Steps
1. Setup: clear profile LOCK; start `node scripts/_profile-cdp.mjs 9222`;
   attach CDP; open `m365.cloud.microsoft/chat` → Settings → Personalization.
2. Pre-flight (0 threads): toggle greyed/forced off = tenant-disabled
   (`enhancedPersonalizationSetting`) → record and ABORT.
3. Save via CDP: fill textarea (native value-setter + input/change), verify
   Save enables, click Save, reopen to confirm persisted. Text is format-only,
   no role override (lane F §5 S0 wording); record the char cap if truncated.
4. Pilot (≤4 threads, marked "cannot conclude"): `scripts/_probe-chat.mjs`
   oneTurn arms — T1 flag-on `magic`; T2 control `magic` (no flags); T3
   flag-on `Claude_Sonnet` (our tool path); T4 full triplet
   (`add_custom_instructions`,`update_memory_plugin`,
   `enable_inferred_memory_read` + `extraAllowed:["MemoryUpdate"]`). Reads:
   ```bash fence, sentinel, `dea_violation`, Disengaged, `contentOrigin`
   (must be DeepLeo), throttle.
5. Replicated runs: ≥3 flag-on/flag-off pairs per question, rotated order;
   tone and triplet questions in SEPARATE runs, never bundled. Budget cap
   ~12 threads total → ONE primary question gets full replicated treatment
   (pilot 4 + 6 = 10); the rest waits for the next rested window. Throttle
   onset = that run INCONCLUSIVE (F24) — wait it out, then resume.
6. Cleanup: clear via CDP (same native-setter + events — plain fill fails to
   enable Save), verify cleared, record final throttle.
7. If the lever works: wire `packages/core/src/session.ts` agent-less
   optionsSets behind `M365_CUSTOM_INSTRUCTIONS=1` (or no flag, if
   unconditional); `bun run build && bun run test:unit`; live E2E
   `M365_DEBUG=1 bun scripts/proxy-verify.mjs --agent --multiturn` with
   `M365_INTENT_VERIFIER_MODEL=bonsai-27b` (LM Studio).

## Acceptance
- Pre-flight outcome recorded (enabled/disabled), zero threads spent.
- Pilot logged explicitly as "cannot conclude" (n=1).
- Replicated runs in `docs/hypotheses.md` with n per arm + evidence pointers.
- Conclusive findings promoted to `docs/m365-copilot-api.md`.

## Evidence
- hypotheses.md §8.4 / H8.14-adjacent + §7 row 09; probe out dir under
  `scripts/`; lane F doc updated with the verdict; ticket Comments.

## Risks
- Filter risk (lane F §4c): instruction text is persistent context on every
  applied turn — abort if flag-on Disengaged > 0 while control is 0, or
  `dea_violation` jumps ≥1 order vs baseline.
- Preview/Frontier volatility: injection point can move silently; re-run one
  pair as a regression canary before relying on the lever.
- Tool calling stays prompt-emulated fenced-shell routing (native tool-
  calling permanently out of scope — license cost); the 8H fail-closed
  verifier and gate order are untouched by this lever.
- Forensic trace: saved instruction is a mailbox artefact — cleanup is
  mandatory; `update_memory_plugin` only in T4 (no chat-driven memory
  writes elsewhere).
- n=1 noise: the pilot never concludes; only replicated pairs count; rotate
  order; total budget ≤12 threads. No PII in captured output — public repo.
