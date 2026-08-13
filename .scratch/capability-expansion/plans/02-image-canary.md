# Plan: Image generation capability canary
> Ticket: .scratch/capability-expansion/issues/02-image-canary.md · Status: ready-for-agent · Blocked by: rested account (PC or laptop; PC auth cache present since 2026-08-10)
## Purpose
Prove the `capabilities:`-array attach path works at all with a visually-obvious image-gen canary (H8.3 `GraphicArt`/flux flags). A clean fire-or-refuse verdict validates every other H8.* capability attach cheaply.
## Preconditions
- Explicit user authorization for live M365 probes; strictly sequential, one thread at a time; ≤12 fresh conv/hr, ≥3 min spacing; hard stop at first empty-503/at-limit.
- Live M365 auth, PC or laptop (`getToken()`, playwright login under `node`). `bun run build` first.
## Steps
1. `bun run build && bun run test:unit`.
2. Agent-less route first (cheapest): `_probe-chat.mjs` oneTurn with `optionsSets:["fluxcopilot","fluxprod","dgencontentv3"]`, prompt "generate an image of a red cube"; watch for an image/blob frame (image payload or adaptive card with an image URL) in the frame stream. n≥3.
3. Capability route: attach `clientOverrides.capabilities:[{name:"GraphicArt"}]` in the `gpts[]` block (same slot as `CodeInterpreter`, §12.6) via `_probe-chat.mjs` with `agentId` set; same prompt; watch for the image frame or a clean refusal.
4. If neither fires, bisect: `variants-bisect.mjs`-style flag sweeps over `fluxcopilot`/`fluxprod`/`dgencontentv3` individually (they may be additive).
5. Record per-config frame evidence (messageType names, image presence) and whether the reply is an image, a refusal, or a hallucinated text description of an image.
## Acceptance
- Canary fires (image/blob frame observed) OR cleanly refuses — evidence of the frame in `docs/hypotheses.md` §8.1.
- Verdict recorded: capability pathway works / pathway dead, with n and exact config.
- No product feature shipped — image-gen stays a probe only.
## Evidence
- `docs/hypotheses.md` §8.1 (H8.3 row + verdict, sample size); frame captures in `scripts/` output dir; ticket ## Comments.
- Promotion to `docs/m365-copilot-api.md` only if conclusive.
## Risks
- Image frames may stream as adaptive cards or blob URLs — dump raw frames, don't judge from text alone.
- Prompt-Shields may refuse image requests shaped like jailbreaks; keep the prompt plain ("generate an image of a red cube").
- Thread throttle on bursts — one conversation per probe, hard stop at first empty-503.
