# Plan: Real server-side code-execution sandbox
> Ticket: .scratch/capability-expansion/issues/01-code-interpreter.md · Status: ready-for-agent · Blocked by: rested account (PC-capable — auth cache present since 2026-08-10; laptop optional)
## Purpose
Prove M365 runs real server-side Python via both routes — H8.1 `optionsSets` flags and H8.2 `CodeInterpreter` capability attach — with frame evidence. If real, promote the config into `packages/core` as a free code-execution tool (highest-payoff ticket: can replace hosted verification cost).
## Preconditions
- Explicit user authorization for live M365 probes. Strictly sequential, one thread at a time; ≤12 fresh conversations/hr, ≥3 min spacing; hard stop at first empty-503/at-limit. An empty reply WITHOUT a `Disengaged` frame = thread throttle, not content filter — report, do not retry.
- Rested account with live M365 auth (PC or laptop); `getToken()` via playwright login run under `node` (connectOverCDP times out under Bun).
- `bun run build` before probes (they import `packages/core/dist`). Never bare `bun test`.
## Steps
1. `bun run build && bun run test:unit`.
2. H8.1 optionsSets probe: `M365_NO_INTERACTIVE=1 node scripts/code-interpreter-probe.mjs` — SHA-256 oracle; optionsSets `["cwc_code_interpreter","cwc_code_interpreter_amsfix","cwc_code_interpreter_citation_fix","code_interpreter_interactive_charts","code_interpreter_matplotlib_patching"]` + extraAllowed `["GeneratedCode","GenerateContentQuery","Progress"]`; plain chat (agentId null). Watch for a `GeneratedCode` frame carrying `hashlib.sha256(...)`; the 64-hex digest must equal the locally computed expected.
3. Control: `... code-interpreter-probe.mjs --control` (no optionsSets) — baseline: does plain magic refuse or hallucinate?
4. Proxy passthrough: `M365_DEBUG=1 bun scripts/proxy-verify.mjs --multiturn` with the hash prompt on the agent-less path — `CODE_INTERPRETER_OPTIONS_SETS` in `packages/core/src/session.ts` is on by default; verify the real digest round-trips as text (`M365_NO_CODE_INTERPRETER=1` toggles off).
5. H8.2 declarative probe: in `packages/core/src/agent.ts::createBot`, attach `clientOverrides.capabilities:[{name:"CodeInterpreter"}]` (the real capability channel per §12.6 decompile) and/or flip `metadata.gptCapabilities.codeInterpreter:true` (documented toggle, currently false); publish; then prompt "run `print(2**100)` in Python" through `_probe-chat.mjs` with `agentId` set; watch for `GeneratedCode` frames vs hallucination.
6. n≥3 per config, fresh conversation per run, order rotated, ≥3 min apart. Save frame dumps per config.
7. If works: make the config the default, `bun run build && bun run test:unit`, spawn the surfacing feature ticket (07) and record in api-doc.
## Acceptance
- Both probes run; `GeneratedCode` frame + correct SHA-256 digest observed per config (n recorded, ≥3).
- Verdict (works / partially / dead) with n in `docs/hypotheses.md` §8.1.
- If works: config promoted in `packages/core`; candidate feature ticket spawned.
## Evidence
- `docs/hypotheses.md` §8.1 (sample size + frames); §8.9 already holds the June-13 confirmation and 2026-08-09 live runs (two distinct sandbox hostnames).
- Frame captures: `scripts/code-interp-out/<ts>/frames.ndjson`; verdict + n in ticket ## Comments.
- Conclusive protocol facts promote to `docs/m365-copilot-api.md` §5.
## Risks
- Content-filter `Disengaged` on tool-shaped prompts (F22) — keep probe prompts plain imperative; a Disengaged frame is NOT throttle.
- Thread throttle: burst → empty-503; honor the hard stop. n=1 noise → ≥3 runs, rotate order.
- Sandbox is M365's, not the harness's: no filesystem/egress guarantees; only use for hashing/math/transforms.
