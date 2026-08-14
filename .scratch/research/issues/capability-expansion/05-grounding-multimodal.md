# 05 — Grounding & multimodal input

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
**Source:** `docs/hypotheses.md` §8.3 H8.9–H8.12

## Goal

Four separate levers, each a cheap canary:

**H8.9** — Web search toggle: `plugins:[{BingWebSearch}]` vs
`optionsSets:["nosearchall"]`; watch `InternalSearchQuery`/sourceAttributions.
**H8.10** — Image INPUT via substrate `UploadFile` POST → `docId` →
`messageAnnotations:[{messageAnnotationType:"ImageFile"}]`.
**H8.11** — Graph/Work grounding via CIQ variants + `at_mention_plugins_enable`.
**H8.12** — Long-doc QA via `optionsSets:["ldqa","ldsummary"]` + File entity.

## Acceptance

- [ ] Each probe run separately; per-H verdict with evidence
- [ ] H8.9: latency + attribution delta measured (off = faster coding answers)
- [ ] H8.10: pixel-level vision confirmed (screenshot → true answer)
- [ ] H8.11/H8.12: grounded citations observed or absent
- [ ] Git-committed verdicts in `docs/hypotheses.md` §8.3

**Out of scope:** uploading a company file without user explicit consent;
changing the default `optionsSets` payload while probes are in flight.

## Comments

- **2026-08-08** — Investigated the stale `image-upload` git worktree/branch
  (`.git/worktrees/m365-copilot-proxy-image-upload`, branch `image-upload` at
  `ce4b14b`). Finding: the working tree is deleted (hollow metadata — no
  `gitdir`, no `logs/HEAD`), and the branch is **fully merged into `main`** (0
  unique commits; `git merge-base main image-upload` = `ce4b14b`, the branch
  head itself). No orphaned work exists. The "image upload" feature intent is
  **this ticket's H8.10** (substrate `UploadFile` → `docId` → `ImageFile`
  message annotation) plus `02-image-canary.md` for image *generation*
  (H8.3 `GraphicArt`/flux). No duplicate ticket created. The dead branch is
  safe to clean up (`git branch -d image-upload` + `git worktree prune`).