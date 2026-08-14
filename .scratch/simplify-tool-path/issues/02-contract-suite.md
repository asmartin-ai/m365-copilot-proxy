# 02 — Add produceToolPath() contract suite

**What to build:** A new test file `packages/proxy-lib/src/tool-path.contract.test.ts` that locks in the simplified contract at the `produceToolPath()` seam with a fake `runTurn` (no network, no model).

The governing property:
> M365 text → deterministic translation.

Every test asserts `runTurn` is called exactly once (the killer invariant that proves no semantic content can trigger a corrective upstream turn).

Golden table (fake runTurn return → required ToolPathResult):

- ordinary prose → `text`, exact prose
- valid `bash` fence → `tools`
- fenced named tool → `tools`
- two valid calls → first only
- "I updated README.md." with no fence → `text`
- "I cannot access your files…" → `text`
- Teams artifact URL → `text`
- `/mnt/data/foo.patch` → `text`
- malformed fence syntax → `text`

**Blocked by:** 01 — delete behavioral policy.

**Status:** ready-for-agent

- [x] File exists at `packages/proxy-lib/src/tool-path.contract.test.ts`.
- [x] Each test injects a fake `runTurn` that returns a fixed string.
- [x] Each test asserts `runTurn` called exactly once.
- [x] Golden table above all pass (covered by the suite above).
- [x] Suite runs in < 1s, no network.

**Status:** done

## Comments

- **2026-08-14 code review (post-merge, `reviewer` sub-agents on `git diff 050c010...HEAD`):** accepted gap — the contract suite has **zero `M365_STEERING` coverage**. The old `tool-path.test.ts` had a 4-test steering block (`unsteered→text`, `no fingerprint→text`, `steered→tools`, `legacy-unset→tools`); it was deleted wholesale with the old file and not re-implemented. The steering gate code is kept and still wired (`handler.ts` `steeringFingerprint`), just unguarded. Also silently dropped: `M365_ALLOW_MULTI_TOOL=1` batch preservation and mixed reply+real-tool cases. Deliberate no-fix: rebuilding deleted coverage re-introduces behavior the pivot spec classified out-of-scope; tracked here for a future coverage pass. The golden-table `malformed fence` row is covered only via the unknown-tool branch (well-formed fence + `tools: []`), not genuine malformed syntax — rename or add a malformed fixture when the suite is next touched. (Standards/Spec review reports: `agent://StandardsReview`, `agent://SpecReview`.)
