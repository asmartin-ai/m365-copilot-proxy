# 05 — Grounding & multimodal input

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** laptop
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