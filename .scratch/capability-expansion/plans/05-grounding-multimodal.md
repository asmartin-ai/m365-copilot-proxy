# Plan: Grounding & multimodal input
> Ticket: .scratch/capability-expansion/issues/05-grounding-multimodal.md · Status: ready-for-agent · Blocked by: rested account (PC or laptop; PC auth cache present since 2026-08-10)
## Purpose
Four cheap canaries on grounding/multimodal levers: web-search toggle (H8.9), image INPUT vision (H8.10), Graph/Work grounding (H8.11), long-doc QA (H8.12) — each decides whether a capability is worth wiring into the proxy.
## Preconditions
- Explicit user authorization for live M365 probes; sequential, one thread at a time; ≤12 fresh conv/hr, ≥3 min spacing; hard stop at first empty-503/at-limit.
- Live M365 auth (PC or laptop); `bun run build` first.
- H8.11/H8.12 need a real OneDrive/company file: EXPLICIT user consent to reference a specific file — never upload or @-reference a company file without it (hard out of scope).
- Do NOT change the default `optionsSets` payload while probes are in flight.
## Steps
1. `bun run build && bun run test:unit`.
2. H8.9 search toggle: write `scripts/search-toggle.mjs` if absent (oneTurn pattern): config A `plugins:[{Id:"BingWebSearch",Source:"BuiltIn"}]` (current default) vs config B `plugins:[]` + `optionsSets:["nosearchall"]`; same fresh-fact query (e.g. a today-dated fact) per config; watch `InternalSearchQuery`/`sourceAttributions` frames; record latency delta (elapsedMs) and attribution presence. n≥3, order rotated.
3. H8.10 image input: write `scripts/image-input-probe.mjs` — substrate `UploadFile` POST (PyRIT shape: `/m365Copilot/UploadFile`, header `X-Variants:feature.EnableImageSupportInUploadFile`) → `docId`/`BlobId`; then attach `messageAnnotations:[{id,messageAnnotationType:"ImageFile"}]` + `optionsSets:["cwcgptvsan",…]` in oneTurn (NOT `entityAnnotationTypes`). Use a local screenshot with a known readable value; ask what it says — pixel-level vision confirmed only if the true answer returns.
4. H8.11 Graph grounding: enable CIQ variants (`feature.EnableLuForChatCIQ`, `feature.enableChatCIQPlugin`) + `optionsSets:["at_mention_plugins_enable"]`; @-reference the consented OneDrive file; watch for grounded citations.
5. H8.12 long-doc QA: `optionsSets:["ldqa","ldsummary"]` + a `File` entity; needle question deep in a long doc; watch for grounded citations and whether the `numLongDocSummary…` counter increments.
6. Per-H verdict (works/partial/dead) with n, latency, and frame evidence.
## Acceptance
- Each probe run separately; per-H verdict with evidence.
- H8.9: latency + attribution delta measured (off = faster, deterministic coding answers).
- H8.10: pixel-level vision confirmed (screenshot → true answer) or cleanly refused.
- H8.11/H8.12: grounded citations observed or absent, with the exact config that produced them.
- Git-committed verdicts in `docs/hypotheses.md` §8.3.
## Evidence
- `docs/hypotheses.md` §8.3 (per-H rows, n, frames); probe output dirs under `scripts/`; ticket ## Comments (incl. 2026-08-08 image-upload branch finding — intent == H8.10, no orphaned work).
- Conclusive facts promote to `docs/m365-copilot-api.md`.
## Risks
- Company-file consent is a hard gate — probes without consent are out of scope by definition.
- `nosearchall` + `plugins:[]` may not fully kill server-side search — verify via absence of `InternalSearchQuery`, not assumption.
- Thread throttle on bursts; one conversation per probe, hard stop at first empty-503. n=1 noise → ≥3 runs, rotate order.
