# 01 — Real server-side code-execution sandbox

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** laptop (source-backed, ~5 msgs)
**Source:** hypotheses §8.1 H8.1 + H8.2

## Goal

Get M365 to run real Python server-side instead of hallucinating results.

**H8.1:** `optionsSets` `["enterprise_flux_work_code_interpreter", "code_interpreter_interactive_charts", "code_interpreter_matplotlib_patching", "codeintfile", "sdretrieval"]` + `allowedMessageTypes` with `GeneratedCode`/`GenerateContentQuery`.
**H8.2:** declarative route — add `capabilities:[{"name":"CodeInterpreter"}]` to the `minimalBots` agent create payload.

Prompt: "run `print(2**100)` in Python"; watch for a `GeneratedCode` frame.

## Acceptance

- [ ] Both probes run; frame evidence captured per config
- [ ] Verdict (works / partially / dead) with n in `docs/hypotheses.md` §8.1
- [ ] If works: promote the config to `integratedas code-execution tool` in
      `packages/core` — candidate feature ticket spawned

## Notes

- Would give the coding agent a **free server-side Python sandbox** (MS docs:
  "available to Copilot Chat users without metered usage").
- If the frame fires, this is the single highest-payoff ticket in the
  effort — it can replace our hosted verification cost.