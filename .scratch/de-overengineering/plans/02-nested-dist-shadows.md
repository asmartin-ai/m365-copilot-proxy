# Plan: Ponytail cleanup pass — final deletion sweep
> Ticket: .scratch/de-overengineering/issues/02-nested-dist-shadows.md (ticket 01 resolved 2026-08-10; this plan is its deferred follow-up) · Status: ready-for-agent · Blocked by: none

## Purpose
Finish the ponytail pass by removing what the 2026-08-10 run left behind:
stale nested duplicate `dist/` copies under `packages/proxy/node_modules`
that shadow the workspace links, plus a fresh zero-caller/stale-reference
audit. Deletion-only; shipped behavior stays byte-for-byte identical. Keeps
the repo lean so a usable agent in pi/Codex builds and tests fast and the
frozen verifier/attestation gating is untouched.

## Preconditions
- Clean tree on `origin/main`. Baseline `bun run build && bun run test:unit`
  green BEFORE any deletion (never bare `bun test` — Bun native runner is not
  the oracle).
- No proxy/agent process running: Windows `mv`/`rm` fail while files are
  locked (the 2026-08-10 deferral reason).
- Out of scope: execution gating (8H verifier, attestation), framing variants
  (already culled), live M365.

## Steps
1. Baseline: `bun run build && bun run test:unit`; record pass count (expect
   ≈290 pass / 3 skip).
2. Dead-export audit: grep `packages/` for the item-1 names
   (`getForcePrompt`, `ForcePromptType`, `withConversation`,
   `FRAMING_VARIANT_NAMES`, `HEALTH_PAYLOAD`) — expect 0 hits (deleted).
   Then sweep every `export ` declaration in `packages/core/src` +
   `packages/proxy-lib/src` against its importers; delete any remaining
   zero-caller export (keep the three force-prompt consts imported by
   tool-path).
3. Duplicate dist copies: inventory
   `packages/proxy/node_modules/@m365-copilot/{core,proxy-lib}` and the
   deeper `…/@m365-copilot/proxy-lib/node_modules/@m365-copilot/core`;
   compare each nested `dist/` to the workspace build (hash or mtime).
   Primary: delete the nested `@m365-copilot` dirs so workspace links
   resolve. Fallback if Windows-locked: refresh the nested `dist/` from the
   workspace builds (documented 2026-08-10 convention) and note the lock in
   the ticket. Verify resolution from `packages/proxy` (e.g.
   `node -e "console.log(require.resolve('@m365-copilot/core'))"` via the
   proxy package) points at `packages/core`, not a node_modules copy.
4. Stale-alias audit: package.json `exports` maps (core, proxy-lib) — drop
   entries whose target file is gone; repo-wide grep for references to
   removed names (`FRAMING_VARIANT_NAMES`, `getForcePrompt`, `compileFull`,
   `FENCED_TOOL_CALL_REGEX`, stale "minimal"-framing probe comments) in
   scripts/docs/tests; fix or delete stale comments/defaults.
5. Rebuild + verify: `bun run build && bun run test:unit` green; `tsc
   --noEmit` per touched package.
6. No live M365 required. Skip the optional interactive-login smoke — item 6
   already landed and passed 2026-08-10.

## Acceptance
- `bun run build && bun run test:unit` green (≈290 pass / 3 skip).
- Zero zero-caller exports remain; no references to any removed name.
- No nested duplicate copies shadow the workspace links (or refreshed and
  documented if Windows locks persist).
- `git diff --stat` shows deletions only; no shipped-path behavior change.

## Evidence
- Ticket `## Comments` (append run summary + lock note); NEXT.md PC-environment
  note updated once removal lands; git diff stats in the commit message.

## Risks
- Windows file locks → fallback refresh path; verify resolution after every
  build or stale dist silently shadows (the 2026-08-10 failure mode).
- `bun test` red-herring: verify with `bun run test:unit` only.
- Public repo: no PII, no machine paths in evidence.
- Do NOT touch the frozen 8H verifier, attestation gate, or fenced.ts
  variants — complexity-only scope.
