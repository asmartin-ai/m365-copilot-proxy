# Plan: Model selection & smart-mode routing
> Ticket: .scratch/capability-expansion/issues/04-model-selection.md · Status: ready-for-agent · Blocked by: rested account (PC or laptop; PC auth cache present since 2026-08-10)
## Purpose
Resolve how the underlying model is selected: `tone` as model selector (H8.6), `ScenarioModels` capability binding (H8.7), and `SwitchRespondingEndpoint` observability into Smart-mode routing (H8.8) — the route to Claude/GPT-5.5 lanes through M365 at zero marginal cost.
## Preconditions
- Explicit user authorization for live M365 probes; sequential, one thread at a time; ≤12 fresh conv/hr, ≥3 min spacing; hard stop at first empty-503/at-limit.
- Rested account with live M365 auth (PC or laptop). `bun run build` before probes.
- Agent-overrides-tone trap: WITH a declarative agent attached, `Claude_Sonnet` routes to GPT-5 (§8.9) — tone probes MUST be agent-less.
## Steps
1. `bun run build && bun run test:unit`.
2. H8.6 tone bisect: run `node scripts/tone-probe.mjs` (or `variants-bisect.mjs --target streaming` pattern) with `agentId:null`; candidates `magic`, `Claude_Sonnet`, `Claude_Sonnet_Reasoning`, `Gpt_5_5_Chat`, `Gpt_5_5_Reasoning`, `Gpt_5_6_Chat`, `Claude_Reasoning`, `Anthropic_Claude`, bogus control. Classify via `contentOrigin`: content = accepted, `BotConnection` deflect or server error = rejected. §8.9/A4 already bisected 9 tones — re-confirm 1x on the rested account, then record; do not re-burn the full sweep.
3. H8.7 ScenarioModels: in `createBot` attach `clientOverrides.capabilities:[{name:"ScenarioModels","models":[{id:"sonnet4-6"}]}]` (guessed id from `cuaAnthropicModels`); publish + chat 1 turn. Even a 400/error body may leak the valid enum — capture the full rejection text.
4. H8.8 SwitchRespondingEndpoint: `_probe-chat.mjs` with `extraAllowed:["SwitchRespondingEndpoint"]`; send a hard reasoning prompt at `tone:"magic"`; log whether the frame fires and any model-id content; repeat with `tone:"Gpt_5_4_Reasoning"` pinned and diff.
5. Verdict per hypothesis (works/partial/dead) with n and exact payloads.
## Acceptance
- H8.6: tone candidates classified valid→content / invalid→fallback via `contentOrigin`; Claude lane status confirmed agent-less.
- H8.7: ScenarioModels probe attempted; error or acceptance captured (leak or bind) — a definitive "not honored" is a valid verdict.
- H8.8: SwitchRespondingEndpoint behavior logged; magic-downgrade detected or ruled out.
- Verdict per hypothesis in `docs/hypotheses.md` §8.2.
## Evidence
- `docs/hypotheses.md` §8.2; frame dumps (origin, messageTypes) in `scripts/` output dir; ticket ## Comments.
- Conclusive tone/capability facts promote to `docs/m365-copilot-api.md`.
## Risks
- No production rebinding to Claude without a user request (out of scope). No tool-format changes.
- Server validates tones: errors are real rejections, not network noise — log status + body verbatim.
- Thread throttle: hard stop at first empty-503; ≥3 min spacing between fresh convs.
