# 01 — Real server-side code-execution sandbox

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
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

## Live evidence (2026-08-09, from agent-less probes)

- **Hosted Python is real and executes truthfully.** On the Claude tone
  (agent-less, `claude-sonnet-4.5`), M365 ran its hosted Python interpreter
  (visible `contentType:"Code"` progress blocks) and returned a REAL result:
  `open("/etc/hostname")` → `SandboxHost-639219344524854762` — a distinct
  sandbox hostname, impossible to fabricate. Two independent runs, two distinct
  sandbox ids (`…344524854762`, `…345700763075`).
- **Runs regardless of the 5 known flags.** The `cwc_code_interpreter*`
  optionsSets were stripped via `M365_NO_CODE_INTERPRETER=1` and the hosted
  Python still ran — the interpreter engages server-side for Claude tones even
  with all known flags removed (possibly another option set or a server-side
  default).
- **Implication for the proxy:** the sandbox answers truthfully but the proxy
  currently passes the result through as plain text — no
  `GeneratedCode`-style frame surfacing, no tool call. The capability exists;
  the surfacing (see ticket 07) is the missing piece.
- Prior oracle: SHA-256 digest verified in `docs/m365-copilot-api.md` §5
  (api-doc, `GeneratedCode` frame with `hashlib.sha256(...)`).