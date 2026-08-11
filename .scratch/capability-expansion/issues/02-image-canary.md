# 02 — Image generation capability canary

**Status:** ready-for-agent
**Category:** enhancement
**Type:** research
**Blocked by:** —
**Source:** `docs/hypotheses.md` §8.1 H8.3

## Goal

`capabilities:[{"name":"GraphicArt"}]` (or the flux flags
`fluxcopilot`/`fluxprod`/`dgencontentv3`) returns generated images over the
WS. Image-gen is a visually obvious **capability-acceptance canary** — it
proves the `capabilities:`-array path works at all, giving weight to every
other H8.*.

Prompt: "generate an image of a red cube"; watch for an image/blob frame.

## Acceptance

- [ ] Canary fires or cleanly refuses (evidence of the frame in
      `docs/hypotheses.md` §8.1)
- [ ] Verdict recorded — pathway works / pathway dead

**Out of scope:** shipping image-gen as a product feature.