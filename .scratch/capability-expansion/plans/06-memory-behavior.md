# Plan: Memory, instructions, behaviour control
> Ticket: .scratch/capability-expansion/issues/06-memory-behavior.md · Status: ready-for-agent · Blocked by: rested account, PC-capable (H8.14 additionally: Copilot plan upgrade)
## Purpose
Persistent/structured behaviour controls that cut per-turn prompt cost and attack the prose-compliance bug from the capability layer: knowledge suppression (H8.13), persistent memory (H8.14 — license-gated), the instructions-blob ceiling (H8.15), and agent delegation (H8.16). Plan the ungated parts (H8.13/15/16) first; H8.14 runs only after the plan upgrade.
## Preconditions
- Explicit user authorization for live M365 probes; sequential, one thread at a time; ≤12 fresh conv/hr, ≥3 min spacing; hard stop at first empty-503/at-limit.
- Rested account with live M365 auth (PC or laptop); `bun run build` before probes.
- H8.14 LICENSE GATE: durable-memory authoring is gated on Basic (NEXT.md — Copilot plan upgrade pending). WS memory flags are dead on Basic (E-O2 pilot + lean-plant retry: 0/3 recall; model self-reports "no ability to permanently store new memories"). Do NOT burn quota re-proving the negative — H8.14 waits for the upgrade, then re-tests (mailbox write → recall). The GUI textarea custom-instructions channel is a separate, working, ungated lever (A1) — not this ticket.
- H8.15 target surface is the `minimalBots` agent `instructions` blob (Copilot Studio 8k limit), NOT the custom-instructions channel (no 8k cap there, A3).
## Steps
1. `bun run build && bun run test:unit`.
2. H8.13: add `behavior_overrides:{special_instructions:{discourage_model_knowledge:true}}` (and `suggestions.disabled:true` as a cheaper parse-canary) to the `createBot` payload; publish; ask a general-knowledge Q the model knows cold (e.g. a well-known fact); verdict = tool-deferral vs answer-from-memory. n≥3 vs a control agent without the flag.
3. H8.15: publish agents whose `instructions` carry a behavioral tail-directive ("ALWAYS end replies with TAILMARK") with the directive at offsets 3.9k / 7.9k / 8.1k / 12k chars (directive padded with inert text); ask a benign question; the highest offset whose TAILMARK echoes = the real cap. NOTE: never use "echo the sentinel" — the model refuses to echo its own instructions (A3); behavioral tail-directives only.
4. H8.16: publish agent B with a sentinel-only instruction ("only answer: SENTINEL-<n>"); create agent A with `worker_agents:[{id:"<TitleId of B>"}]`; ask A something only B knows; pass = B's answer arrives through A's thread.
5. H8.14 (post-upgrade, gated): retry `node scripts/memory-channel-probe.mjs` lean-plant recall (sakura→photovoltaics) with WRITE+READ flags; pass = fresh-conversation recall of the planted target.
6. Per-H verdict (works/partial/dead) with n in `docs/hypotheses.md` §8.4.
## Acceptance
- H8.13: general-knowledge Q answered with tool-deferral vs memory — diff vs control agent recorded.
- H8.15: highest echoed tail-directive offset = the real cap (behavioral test, not echo).
- H8.16: agent B answers through A's thread.
- H8.14: executed ONLY after the plan upgrade; "remember sakura" survives a NEW conversation, or a clean negative with the upgrade in place.
- Per-h result in `docs/hypotheses.md` §8.4.
## Evidence
- `docs/hypotheses.md` §8.4 + §8.5 A1/A3 (channel facts); `scripts/memory-channel-probe.mjs` output; agent publishes logged in ticket ## Comments.
## Risks
- Directive-heavy "remember exactly / do not mention" plants are jailbreak-shaped (F22) → Disengaged; use lean declarative phrasings only.
- H8.15: guardrail refusal to echo instructions is expected — behavioral markers only; pad with inert text so the directive sits at the target offset.
- Don't waste quota on H8.14 pre-upgrade (0/3 already proven); thread throttle hard stop applies to every probe.
