# Plan: Passive throttle telemetry — decision gate
> Ticket: .scratch/fallback-lane-telemetry/issues/02-decision-gate.md (ticket 01 implementation resolved 2026-08-09; this plan closes the spec's open decision gate) · Status: ready-for-agent · Blocked by: ≥1 week of real use OR ≥3 throttle episodes (whichever first)

## Purpose
Close the fallback-lane decision gate: confirm the shipped telemetry is live
and passive (zero extra M365 traffic), collect ≥1 week of real-use data (or
≥3 throttle episodes), then apply the spec decision rule — frequent long
lulls → open the Option-2 local fallback-lane build effort; rare/short lulls
→ close Option 2 as `wontfix` with the numbers. Gives the "usable agent in
pi/Codex" goal a data-driven answer on whether a local continuity lane is
worth its complexity.

## Preconditions
- Telemetry implementation already shipped (2026-08-09): NDJSON appended to
  `~/.config/opencode-m365/throttle-telemetry.ndjson`
  (`M365_THROTTLE_TELEMETRY_FILE` overrides; `M365_NO_TELEMETRY=1` disables
  writes but keeps /health counters). File stays OUTSIDE the public repo.
- 2026-08-10 quarantine: the old log was 100% unit-test pollution; moved to
  `~/.config/opencode-m365/quarantine-2026-08-10/`. The real-use log starts
  from the quarantine date; vitest.setup.ts prevents new test writes.
- No live-probe authorization needed — this plan generates ZERO M365 traffic
  (passive observation of normal proxy use only).

## Steps
1. Verify log state: line count + event-type histogram of the live file;
   assert no line has `convIdHash == sha256("handler-conversation")` (the
   unit-test tell). Verify `M365_NO_TELEMETRY` is NOT set in the proxy env.
2. Verify passivity: confirm emission sites are the existing detection points
   only — `noteRequestOutcome` (core/auth-recovery.ts: empty-throttle +
   backoff-enter/exit) and handler.ts (disengaged, at-limit) — and no
   telemetry path issues a fetch/request. `bun run build && bun run test:unit`
   green (baseline 239 pass / 3 skip; never bare `bun test`).
3. Collect: normal proxy use for ≥1 week from the quarantine date OR until
   ≥3 throttle episodes are logged — whichever first. No probes, no loops,
   no log rotation/deletion mid-gate.
4. Analyze at the gate: parse the NDJSON — per-event counts, episodes
   (contiguous empty-throttle / backoff-enter clusters), backoff durations
   from enter/exit pairs, at-limit `current/max`, disengaged framing
   distribution; summarize per-day lull frequency + duration.
5. Apply the spec decision rule: frequent long lulls → open the fallback-lane
   build effort (two-stage bake-off per research doc §8 item 2, including the
   judge-required prompt-injection threat model); rare/short lulls → close
   Option 2 as `wontfix` with the numbers.
6. Record the verdict + stats in `docs/hypotheses.md` (extend the §17
   research-graduation entry) and in the ticket `## Comments`; close the
   ticket with the decision.

## Acceptance
- Live log shows ≥1 week of real use or ≥3 episodes, zero test-pollution
  lines, and no M365 traffic attributable to the telemetry path.
- Decision recorded per the spec rule, backed by the log numbers.
- `M365_NO_TELEMETRY=1` confirmed to stop writes (covered by unit tests).

## Evidence
- `~/.config/opencode-m365/throttle-telemetry.ndjson` (out-of-repo — never
  commit), `docs/hypotheses.md` §17, ticket `## Comments`.

## Risks
- Test-pollution recurrence (check the convIdHash tell at every read).
- `M365_NO_TELEMETRY=1` accidentally left set → empty log → wrong "no lulls"
  verdict; check env before the gate read.
- Writes are best-effort (log.error only, never throws) — a write failure
  silently starves the log; verify the file grows during the window.
- Log rotation/deletion resets the window; keep the file untouched.
- Public repo: never commit the log, raw convIds, or quota numbers tied to
  accounts.
