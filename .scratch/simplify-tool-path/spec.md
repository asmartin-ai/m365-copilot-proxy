# Spec: simplify tool-path — translate, do not infer

## Problem

The proxy has grown a behavioral-correction layer on top of a translator. In
`packages/proxy-lib/src/tool-path.ts` (`produceToolPath()`), the single seam
that turns a buffered M365 response into an OpenAI response currently also:

1. runs a local LLM (8H / Bonsai) to second-guess whether prose is "worth
   executing,"
2. runs a client-attestation gate to bypass that local LLM,
3. classifies model output via regexes ("I can't access your files", "I've
   updated README.md") and **forces another M365 turn**,
4. infers read-only tool calls the model never emitted,
5. escalates through confab / hallucination / remote-artifact force-prompts.

That is an **agent behavior-policy layer**, not a protocol adapter. The user
and the architectural review (GPT-4o, conversation "CopilotAgent — Project
Complexity Assessment") agree: this is accidental complexity and the project has
blown past its scope.

## Solution

One simplification branch. In `produceToolPath()`, delete the four
behavioral mechanisms above and leave the **deterministic translation** and
**M365-protocol resilience** untouched. Enforce the new contract with a
machine-testable suite at the `produceToolPath()` seam — no network, no model,
no quota.

The hypothesis: removing behavior-correction does not regress real agentic
score (the bench), because the harness (pi/OMP/Codex) owns authorization,
prompting, and retry policy. If the bench stays green, the complexity was not
earning its keep.

## Root-cause location

`packages/proxy-lib/src/tool-path.ts` — `produceToolPath()`. The file already
exhibits a clean seam: the decision logic is the only non-transport code.
GPT-4o's characterization ("lines 82–206") was approximate; the real spans
(from a 2026-08-13 read):

- Imports for deletion: `readOnlyFallbackToolCall` (L15), `looksLikeConfabulation`
  + `looksLikeHallucinatedCompletion` + `looksLikeRemoteArtifactCompletion` (L11–13),
  `IntentVerifier` / `AttestationGate` / `AttestationClient` types (L20–21).
- Read-only fallback: L56–61.
- Confab / hallucination / remote-artifact forced-retry loop: L89–133.
- Post-retry fail-closed 502 for "file mutation without tool call": L148–155.
- Attestation + 8H verifier gate (the whole `attested` / `intentVerifier`
  branch): L184–203.
- Force-prompt imports (`CONFAB_FORCE_PROMPT`, `HALLUCINATION_FORCE_PROMPT`,
  `REMOTE_ARTIFACT_FORCE_PROMPT`): L16–18 — also delete once the loop is gone.

## What stays (the translator)

- `parseToolCalls()` — fence → tool_calls. **Translation.**
- `isProseDocument()` — document guard. **Translation** (prevents markdown
  documents with code fences from being executed as shell).
- One-call-per-turn (`M365_ALLOW_MULTI_TOOL`): compatibility policy, not
  behavior. **Keep.**
- Steering-attribution gate (`M365_STEERING`): custom-instructions transport —
  protocol adaptation, not intent guessing. **Keep** (it routes on fingerprint,
  not semantics).
- `reply` → text conversion: representation. **Keep.**
- `markSent`, `registerToolCalls`: bookkeeping. **Keep.**

## What goes (policy the proxy must not make)

- Local LLM intent verifier — the client decides what counts as executable.
- Client attestation — the client signs; this is the harness's job.
- Read-only fallback inference — no invented tool calls.
- Confab / hallucination / remote-artifact retry-on-semantics — no second M365
  turn driven by English interpretation.
- The 502 "file mutation without tool call" detector — that too is semantics.

## User stories

1. As an OpenAI agent, I want a valid bash fence to arrive as a tool_call, so
   that nothing inspects whether I "really meant" it.
2. As a maintainer, I want `runTurn` called exactly once per input, so that no
   classified response can trigger a corrective upstream turn.
3. As a developer, I want obsolete tickets marked `wontfix` up front, so I
   never waste a turn on a killed axis.
4. As a tester, I want `produceToolPath()` to fail its suite if any classifier
   fires, so regressions are red until explicitly re-approved.
5. As a reviewer, I want zero new flags or modes, so the branch is a hypothesis,
   not another toggle.

## Acceptance criteria

1. No production import of `IntentVerifier`, `AttestationGate`, or
   `AttestationClient` **on the runtime path** (`tool-path.ts`, `handler.ts`,
   `responses.ts`, `index.ts` routing). The standalone `attestation.ts` module
   + its unit test + `handleAttestationRequest` endpoint + public re-exports
   are preserved as research artifacts (not on the runtime path), per GPT-4o:
   "delete the integration tests, preserve the module."
2. `produceToolPath()` contains no call to a local model, HMAC, or any
   classification of prose semantics.
3. A single injected `runTurn` mock is the only way the seam talks to M365
   (property tested).
4. Existing deterministic translation tests still pass (parse, render,
   one-call-per-turn, steering gate, reply conversion).
5. `scripts/bench` is unchanged and still green against current `main` behavior.
6. No new configuration, mode, or compatibility flag introduced.

## Seam & contract test

Suite: `packages/proxy-lib/src/tool-path.contract.test.ts`
(renamed / new — the existing `tool-path.test.ts` covers the old behavior; the
contract suite lives alongside and pins the simplified behavior).

`produceToolPath()` accepts an injected `runTurn: (prompt) => BufferedTurn`.
Each test injects a **deterministic fake** that returns a fixed string and
asserts the resulting `ToolPathResult`. The killer invariant is asserted in
every test:

> For every input, `runTurn` is called exactly once.

Golden table (fake M365 text → required result):

| Fake M365 output | Result |
|---|---|
| ordinary prose | exact prose (`text`) |
| ` ```bash ` fence with a `bash` tool | `tools` (one call) |
| fenced named tool | `tools` |
| two valid calls | first only (one-call-per-turn) |
| "I updated README.md." with no fence | `text` |
| "I cannot access your files…" | `text` |
| Teams artifact URL | `text` |
| `/mnt/data/foo.patch` | `text` |
| malformed fence syntax | text / parser-defined non-tool |

## Tracker changes

Per GPT-4o: **rewrite in place**, do not make a parallel `simplify/` track.
The `.scratch/execution-intent-verifier/`, `.scratch/client-attested-execution/`,
and `.scratch/fallback-lane-telemetry/` tickets contradict the pivot and
become `wontfix` with reason: *"Superseded by architecture pivot: the proxy
translates observable M365 output; execution intent/policy belongs to the
consuming harness."*

Active pivot tickets (3–5):

1. `simplify-tool-path` — the deletion branch + contract suite (this spec).
2. `wontfix-survey` — reclassify the 12 scratch feature dirs against the new
   boundary (keep vs move-to-research vs wontfix).
3. `test-retention-audit` — apply the keep/delete rule to the 346/unit suite.

## Out of scope

- `formatMessages()` simplification (the tool-definition injection lives
  there; a separate pass).
- Package restructure (`packages/m365` / `server`).
- Moving the injection ladder to `research/` (that is the `wontfix-survey`
  ticket's job).
- Live M365 verification (quota-gated).

## Further notes

- The 8H frozen verifier corpora, the held-out eval, and the experiment
  harness under `experiments/tool-decision/` are **research artifacts**, not
  product architecture. They stay on disk as evidence, not in the runtime.
- Auth, M365 WS/SignalR, agent creation/resolution, tone routing, session pool,
  delta context, scheduler, throttle/backoff (transport-level), stale-agent
  refresh, Disengaged handling, SSE, images, schemas — all untouched.
