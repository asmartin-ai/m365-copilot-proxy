# 06 — Memory, instructions, behaviour control

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** laptop
**Source:** `docs/hypotheses.md` §8.4 H8.13–H8.16

## Goal

Persistent / structured behaviour controls that reduce per-turn prompt cost:

**H8.13** — `behavior_overrides.special_instructions.discourage_model_knowledge:true` suppresses base-model knowledge, prefers tools (the compliance lever attacked only via prompt wording so far).
**H8.14** — `optionsSets:["add_custom_instructions","update_memory_plugin","enable_inferred_memory_read"]` opens a persistent memory channel (persona survives turns without re-sending).
**H8.15** — the `instructions` blob has a hard **8,000-char server ceiling** and silently truncates — find our real cap with sentinels at 3.9k/7.9k/8.1k.
**H8.16** — `worker_agents:[{id:<TitleId>}]` lets one published agent delegate to another (multi-agent composition).

## Acceptance

- [ ] H8.13: general-knowledge Q answered with tool-deferral vs memory
- [ ] H8.14: "remember sakura" survives a NEW conversation
- [ ] H8.15: highest *echoed* sentinel offset = the real cap
- [ ] H8.16: agent B answers what only B knows via A's thread
- [ ] Per-h result in `docs/hypotheses.md` §8.4

**Out of scope:** H8.15 cap-fix — knowing the cap is the deliverable, not changing the editor.