# Fallback Lane Telemetry

**Status:** ready — decision gate for research Option 2 (local fallback lane)
**Source:** `docs/research/2026-08-09-local-models-8gb-blackwell.md` §7
Option 2 · `docs/research/notes/judge-review.md` (findings 8 + risk list)

## What this effort is

Passive measurement only. Option 2 — a local fallback lane serving continuity
during M365 throttle lulls — is worth its complexity only if throttle lulls
are frequent and long enough to matter. This effort logs degradation events
during **normal** proxy use. **Zero extra M365 traffic: no new requests, no
probes, no loops.** User constraint honored: "as long as we don't hit M365 too
much too fast."

## Decision rule

After ≥1 week of real use (or ≥3 observed throttle episodes, whichever first),
review the telemetry log:

- Frequent, long lulls → open the fallback-lane build effort (two-stage
  bake-off per research doc §8 item 2, including the prompt-injection threat
  model the judge requires).
- Rare or short lulls → close Option 2 as `wontfix` with the numbers.

## Ticket map

| # | Ticket | Blocked by | M365 traffic |
|---|--------|------------|--------------|
| 01 | Passive throttle telemetry | — | 0 (passive) |
