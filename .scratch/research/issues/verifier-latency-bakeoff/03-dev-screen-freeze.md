# 03 — Dev-corpus screen, freeze ONE candidate

**Status:** resolved (CLOSED-REJECTED — five candidates unsafe FP on ambiguous-002; Ministral-3-3B failed selective accuracy; no candidate frozen, ticket 04 ineligible; NEXT.md. Status line was stale; corrected 2026-08-11)
**Category:** enhancement
**Blocked by:** 01, 02

## Context

Screen replacement candidates on the 28-case DEV corpus and freeze exactly one
model + configuration for the held-out gate. Comparing candidates on the
held-out split is forbidden (ADR-0002; judge finding 2).

**Candidates:** Bonsai-27B thinking-off (via ticket 02 kwargs), Bonsai-8B
thinking-off, Ministral-3-3B-Instruct (no thinking mode — true env swap),
Nemotron-3-Nano-4B, Gemma 4 E2B (QAT), Qwen3.5-4B, LFM2.5-1.2B-Instruct.
Include ticket 01's logprob-scorer variant if it passed.

**Existing dev evidence (do not re-run):** qwythos-9B 0 unsafe FP / selAcc
0.808; Qwen3.5-4B 0 unsafe FP / selAcc 0.692 (TEXT-biased); LFM2.5-2.6B
DISQUALIFIED (2 stable unsafe FPs).

## Change

1. Download/serve each candidate on the laptop (official GGUFs where they
   exist; record file hash + quant per candidate).
2. Run each on the 28 dev cases: temp 0, identity-guard the echoed `model`
   field (LM Studio footgun, NEXT.md), budget ≥2048 max_tokens for models
   that still emit `reasoning_content`.
3. Early-exit any candidate with ≥1 unsafe FP — do not burn its remaining cases.
4. From survivors, freeze ONE model + configuration (weights hash, quant,
   template kwargs, max_tokens, temperature). Write
   `experiments/tool-decision/execution-intent/results/bakeoff-freeze.json`.

## Acceptance

- [ ] Per-candidate dev numbers (unsafe FP, selAcc, exeRec, txtRec, coverage,
      median latency) logged in `docs/hypotheses.md` with n=28 each
- [ ] Exactly one freeze artifact committed naming model + full configuration
- [ ] Held-out cases NOT touched (validate-split confirms)
- [ ] Zero M365 traffic

## Comments
