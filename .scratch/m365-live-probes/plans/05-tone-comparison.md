# Plan: 05 — Tone comparison (generalisation of F2–F4)
> Ticket: .scratch/m365-live-probes/issues/05-tone-comparison.md · Status: ready-for-agent · Blocked by: none
## Purpose
Repeat the compliance A/B across the `MODEL_TONES` wire tones to test whether F2–F4 (measured on `magic`) generalise off `magic`. Answers whether the fenced-format/shell-routing win is tone-specific — decides which tones the proxy should route agents to, and which need prompt-emulated tool-calling workarounds.
## Preconditions
- Explicit user authorization for live M365 probes; rested account (backoff 0, fresh cache 2026-08-10 verified).
- Cost ~50 msgs: 5 prompts × ~8-10 tones at n=1, plus confirm reps → ≥2 session days at ≤12/hr, ≥3 min spacing.
- Canonical wire tones from `packages/core/src/copilot.ts` `MODEL_TONES` values: `magic`, `Gpt_Quick`, `Gpt_Reasoning`, `Claude_Sonnet`, `Claude_Sonnet_Reasoning`, `Claude_Opus`, `Gpt_5_6_Reasoning`, `Gpt_5_5_Chat`, `Gpt_5_5_Reasoning`, `Gpt_5_4_Reasoning`, `Gpt_5_4_Quick`, `Gpt_5_3_Quick`, `Gpt_5_3_Reasoning`, `Gpt_5_2_Quick`, `Gpt_5_2_Reasoning`. Server rejects unknown tones (`Failed to invoke 'Chat'`) — skip unverified/dead tones (DeepLeo/BotConnection trap).
## Steps
1. Authorize; confirm rested account.
2. Write `scripts/tone-comparison.mjs` (generalise `tool-compliance-experiment.mjs`: same 5 compliance prompts, sweep a `--tone` param instead of variants) reusing `_probe-chat.mjs` (`o.tone`, default `"magic"`).
3. First pass, n=1 per tone, order rotated across tones, cost-bounded subset in info-gain order: `magic` (baseline) → `Claude_Sonnet` (agent-less path) → `Gpt_Quick` / `Gpt_Reasoning` → `Gpt_5_6_Reasoning` → remainder per budget. Record per tone: SOLVED rate, outcome mix, `scores.dea_violation`.
4. Confirm any non-magic tone that beats or ties `magic` with `--repeat 3` (order rotated across reps).
5. Run with pacing: `M365_NO_INTERACTIVE=1 CHROMIUM_PATH=$(which chromium) bun scripts/tone-comparison.mjs`; hard stop at first empty-503/at-limit; resume next rested window.
6. Write the generalisation verdict in `docs/hypotheses.md` (F2–F4 rows: confirmed-on-which-tones / refuted / inconclusive with n); append ## Comments to the ticket.
## Acceptance
- Compliance run per tone on a rested account (n stated per tone).
- Per-tone SOLVED rate + outcome mix with n.
- Generalisation verdict logged in `docs/hypotheses.md`.
## Evidence
- `scripts/tone-comparison-out/<TS>/results.json`; `docs/hypotheses.md` F2–F4 + §7 row; ticket ## Comments.
## Risks
- 15 tones × 5 prompts exceeds the ~50-msg budget at n=1 — the subset choice is explicit; unpicked tones stay "untested". Tone latency varies hugely (`*_Reasoning` 10–30s) — budget wall-clock. Compliance is prompt-shape-sensitive; keep the exact F2–F4 prompts. Reasoning tones may refuse tool-framing differently — classify rejections vs Disengaged vs throttle (empty without Disengaged frame = throttle).
