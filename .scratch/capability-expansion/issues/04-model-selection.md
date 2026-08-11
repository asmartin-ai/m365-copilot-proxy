# 04 — Model selection & smart-mode routing

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
**Source:** `docs/hypotheses.md` §8.2 H8.6, H8.7, H8.8

## Goal

Three related probes on selecting/routing the underlying model:

**H8.6** — `tone` accepts a Claude value (`Claude_Sonnet`, `Anthropic_Claude`, …) and newer `Gpt_5_5_*`. `tone` may *be* the model selector.
**H8.7** — `capabilities:[{"name":"ScenarioModels","models":[{id}]}]` is a back-door model binding for `minimalBots` agents (guessed id `sonnet4-6`; even a rejection error may leak the enum).
**H8.8** — `SwitchRespondingEndpoint` in `allowedMessageTypes` reveals mid-stream "Smart mode" routing (detect `magic` downgrades to the fast model).

## Acceptance

- [ ] Tone candidates bisected via `variants-bisect.mjs`; valid → content, invalid → fallback detected via `contentOrigin`
- [ ] ScenarioModels probe attempted; error/acceptance captured (leak or bind)
- [ ] SwitchRespondingEndpoint behaviour logged
- [ ] Verdict per hypothesis in `docs/hypotheses.md` §8.2

**Out of scope:** rebinding production to Claude without a user request; tool-format changes.