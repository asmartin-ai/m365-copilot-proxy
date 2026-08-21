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

## Related research (2026-08-14, updated 2026-08-21)

Local-model candidates for the fallback lane. Original candidate:
Nemotron REAP-20B (11.5 GB IQ4_NL, 3B active, native tool calling,
1M context) — `docs/research/notes/nemotron-3.5-lightning-variants.md`.

**2026-08-21 landscape update:** two stronger candidates shipped since —
Qwen3.8-27B (dense, quality ceiling for local coding agents) and Meta
Muse Glimmer 30B (Apache 2.0, agent-native, DFlash speculative decoding).
See the note's §7 addendum for the revised four-candidate bake-off table.
The REAP-20B download is ON HOLD until this effort actually opens the
bake-off; if it opens, screen all four per §7.

**Telemetry status (2026-08-21):** no
`~/.config/opencode-m365/throttle-telemetry.ndjson` on the PC — zero
events collected; the proxy has not run on this machine since ticket 01
landed. Decision gate needs real proxy use (PC or laptop) to accumulate
data before the ≥1 week / ≥3 episodes review can run.
