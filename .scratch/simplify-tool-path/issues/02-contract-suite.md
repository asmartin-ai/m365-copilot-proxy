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
