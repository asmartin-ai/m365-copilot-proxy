# 01 — Ponytail cleanup pass (de-over-engineering)

**Status:** resolved (2026-08-10)
**Category:** enhancement
**Type:** code
**Blocked by:** —

## Goal

Cut ~400 lines of dead code, duplicated logic, and concluded-experiment
machinery flagged by the 2026-08-10 ponytail review. Shipped behavior stays
byte-for-byte identical: every item is a deletion or a refactor with the
existing unit tests as the oracle. Nothing requires live M365 except the
optional item 6 smoke.

## Required code surface

1. **Zero-caller dead exports — delete.**
   - `getForcePrompt()` + `ForcePromptType` (`packages/proxy-lib/src/force-prompts.ts:35`).
     tool-path.ts already inlines the same three-detector selection. Keep the
     three prompt consts (`CONFAB_FORCE_PROMPT` etc.), which tool-path imports.
   - `SessionPool.withConversation()` (`packages/proxy-lib/src/session-pool.ts:77`).
     The handler calls `acquire()` + `resolve()` separately.
   - `FRAMING_VARIANT_NAMES` (`packages/core/src/fenced.ts:529`). Exported, never referenced.
   - `HEALTH_PAYLOAD` (`packages/proxy-lib/src/index.ts:81`). Dead const; `buildHealthPayload` builds its own body.

2. **Cull the measured-loser framing variants** in `FRAMING_VARIANTS`
   (`packages/core/src/fenced.ts`, roughly 262-520). Delete `minimal`,
   `recency`, `proof_demand`, `persona`, `react`, `negative`, `terse`.
   F18 verdicts (docs/hypotheses.md): persona 0/5, proof_demand 1/5,
   recency/react 2/5, terse/negative/minimal 2/4; baseline ≈ fewshot best.
   Keep: `baseline`, `softened` (live F22 retry path), `demo_only`,
   `session_facts`, `fewshot`.
   - Cutover: three probes default to `minimal` — update to `baseline`:
     `scripts/claude-tools-probe.mjs:8`, `scripts/m365-gui-emulate.mjs:10`,
     `scripts/disengage-agentless-probe.mjs:10`.
   - Cutover: sweep STRATS lists — `scripts/bench/analyze-sweep.mjs:15`,
     `scripts/bench/overnight-sweep.sh:34`, `scripts/bench/sweep2.sh:19`,
     `scripts/bench/variant-sweep.sh:20-30`.
   - Unknown-variant fallback already resolves to `baseline`; unchanged.

3. **Collapse duplicated logic.**
   - `boundedDeltaResult` (`packages/proxy-lib/src/context-compiler.ts:20`) is a
     verbatim copy of `boundedToolResult` (`packages/core/src/tools.ts:151-162`),
     same env var. Export one, import in both.
   - `renderFencedTemplate` (`packages/core/src/fenced.ts:176`) re-implements
     `renderFencedCall` (`fenced.ts:161`) with `<h>` placeholder args. One
     function, value-source parameter.
   - `parseToolCalls` (`packages/core/src/tools.ts`): the legacy ```tool_call
     fallback loop duplicates the JSON-fallback loop verbatim. Run the JSON
     regex over the fenced parse's `leftover` (unknown info-strings already
     survive there) and delete the second loop + `FENCED_TOOL_CALL_REGEX`.

4. **Drop the ContextCompiler ceremony** (`packages/proxy-lib/src/context-compiler.ts:27-44`).
   One implementation, and `compileFull` is a passthrough to `formatMessages`.
   The handler calls `formatMessages` directly; the module keeps only
   `compileDelta` + `LOCAL_TOOL_REMINDER`.

5. **Intent-verifier: delete the drift map only** (`packages/proxy-lib/src/intent-verifier.ts:136-140`).
   Log-only, zero behavioral effect. Keep the LRU cache — ticket
   `.scratch/execution-intent-verifier/issues/01-live-validation.md` demonstrated
   real byte-identical-planner-text hits (deterministic 10A offline hit +
   persistent live two-turn run), so the cache is not dead weight. `inflight`
   single-flight is borderline; keep unless proven dead.

6. **[Optional, risky] Replace the hand-rolled CdpClient with playwright**
   (`packages/core/src/auth.ts:68-150`). playwright is already a dependency in
   this file (`chromium.executablePath()`); `launchPersistentContext` +
   `page.on("request")` scraping the nativeclient auth code removes ~120 lines.
   Only for a session that can run a real interactive login; otherwise defer —
   auth is the most fragile surface in the repo.

## Acceptance criteria

- [ ] `bun run build && bun test` green; no behavior change to shipped paths.
- [ ] No zero-caller exports remain in the four files from item 1.
- [ ] Registry has exactly five variants; unknown-variant fallback still resolves
      to `baseline`; probe defaults and sweep STRATS lists updated.
- [ ] One bounded-truncation function exists; `renderFencedTemplate` is derived
      from `renderFencedCall`; `FENCED_TOOL_CALL_REGEX` gone while stray-JSON
      tolerance still works (fenced/tools tests pass).
- [ ] Handler compiles prompts without `contextCompiler.compileFull`.
- [ ] intent-verifier tests pass with the drift map removed.
- [ ] (item 6 only) Interactive-login smoke: `M365_ENABLE_INTERACTIVE_APPROVAL=1`
      flow scrapes the nativeclient code and acquires a token.

## Out of scope

- Correctness/security fixes (attestation gate, 8H fail-closed core) — the
  review was complexity-only.
- The confab/hallucinated/remote-artifact detector regexes (empirically
  load-bearing, bench-validated).
- Anything needing live M365 beyond the item-6 login smoke.

## Validation

```sh
bun run build
bun test
```

Item 6 additionally requires the interactive-login smoke (see acceptance).

## Comments

- Source: ponytail review 2026-08-10 (net ≈ -400 lines).
- Finding 5 softened from "delete cache + drift + inflight" after reading
  `.scratch/execution-intent-verifier/issues/01-live-validation.md` — the LRU
  cache has live-validated byte-identical-text hits; only the log-only drift
  map is cut.
- 2026-08-10 resolved. Items 1-5 landed; net −250 lines (+96/−357 across 16
  files, plus one regression test). Item 6 (CdpClient → playwright) deferred
  per its own condition: it needs a session that can run a real interactive
  login, and auth is the most fragile surface in the repo.
- Verification: `bun run build && bun run vitest run` = 290 pass / 3 skip
  (the 3 skips are the `M365_LIVE`-gated live suite) — identical to pristine
  HEAD (289/3) plus the one new name-key tolerance test. Adversarial review
  (reviewer agent) on the diff found one real tolerance regression — the
  legacy ```tool_call + `{"name":…}` JSON path dropped with
  `FENCED_TOOL_CALL_REGEX` — fixed by broadening `TOOL_CALL_REGEX` to
  `(?:tool|name)` + a regression test; its other findings were stale doc
  references (COMPONENT_REFERENCE.md, laguna-slice1-prompt.txt), a stale
  "minimal framing" probe comment, and docs wording, all updated.
- Known intended delta vs the ticket's "byte-identical" goal (reviewer
  finding 2, matches ticket item 3's own instruction): the fenced path no
  longer early-returns, so a stray `{"tool":…}` JSON beside a fenced call is
  now parsed as an additional call (handler keep-first drops it unless
  `M365_ALLOW_MULTI_TOOL=1`), `{final}`/`{confidence}` objects beside fenced
  calls are stripped instead of unwrapped, and empty fence pairs are removed
  from fenced-path textContent. No test pinned the old behavior.
- The `claude-tools-probe.mjs` "minimal" default the ticket listed did not
  exist — it already defaulted to `softened` (kept); only the other two
  probes were flipped.
- Note: `bun test` runs Bun's native runner, which is NOT the project oracle
  (it reports 20 fails/11 errors even on pristine HEAD — vitest-API
  incompatibilities). The canonical suite is `bun run test:unit` /
  `bun run vitest run`.
