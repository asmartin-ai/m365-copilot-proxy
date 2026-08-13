# Plan: Leverage M365 built-in tools as proxy features (websearch, images, hosted code execution)
> Ticket: .scratch/capability-expansion/issues/07-m365-builtin-tools-feature.md · Status: backlog · Blocked by: rested account (PC or laptop); NOT before tickets 01/02 verdicts
## Purpose
Surface M365's real built-in capabilities — web search (Bing plugin), image generation, hosted Python sandbox — to OpenAI-compatible clients as tools or structured attachments. The capability exists and is already invoked (model self-reports `search_web`, `web_fetch`, `python_execution`, `image_gen`; 2026-08-09 steering probe); the proxy currently passes everything but the hosted-shell shape through as plain text. This ticket is proxy-side surfacing, not capability acquisition.
## Preconditions
- BLOCKER: live M365 auth (PC or laptop) + rested account, AND verdicts from tickets 01 (code-execution sandbox) and 02 (image canary) must land first — this plan bundles their survivors. Do not start before both verdicts exist.
- Explicit user authorization for live M365 probes; sequential, one thread at a time; ≤12 fresh conv/hr, ≥3 min spacing; hard stop at first empty-503/at-limit.
- Native tool-calling stays permanently OUT OF SCOPE (AGENTS.md §8.11): no MCP/Dataverse/Studio-license paths. Tool calling remains prompt-emulated fenced-shell routing; this ticket maps M365's built-in frames onto OpenAI-compatible shapes, nothing more.
- `bun run build` before any code work; browser/login scripts under `node`.
## Steps
1. Land 01/02 verdicts first (their plans); re-read this ticket's acceptance against the verdicts.
2. Per-built-in probe verdicts with sample size, reusing existing evidence where conclusive:
   - Search: H8.9 (search-toggle probe; `InternalSearchQuery`/`sourceAttributions` frames) — plugin already sent agent-less.
   - Images: H8.3 (GraphicArt canary) + H8.10 (image input) verdicts from ticket 02/05.
   - Code exec: H8.1/H8.2 (SHA-256 oracle, `GeneratedCode` frames) — real and truthful already (2026-08-09: two distinct sandbox hostnames).
3. Decide per built-in: surface as (a) a tool, (b) structured attachment, or (c) keep text passthrough. Draft mappings: `web_search` tool call → Bing plugin turn, results as `InternalSearchQuery`/`sourceAttributions` → tool output with citations; image frames → OpenAI image output shape; `GeneratedCode` frames → sandboxed `execute_python` tool (real execution, no local-shell routing).
4. Implement in `packages/core` (session/native-actions side) + `packages/proxy-lib` (OpenAI-facing shape) behind an opt-in flag; default path unchanged.
5. `bun run build && bun run test:unit` (never bare `bun test`).
6. Live E2E: `M365_DEBUG=1 bun scripts/proxy-verify.mjs --agent --multiturn` exercising the new tool/attachment surfaces; verify the client-visible shape, not just raw frames.
7. Record the decision matrix (tool / attachment / passthrough per built-in) and spawn concrete feature ticket(s) with the mappings.
## Acceptance
- Each built-in (search / images / code exec) has a probe verdict in `docs/hypotheses.md` with sample size.
- Decision recorded: which surface as tools, which as attachments, which stay text passthrough.
- Feature ticket(s) spawned with concrete OpenAI-compatible mappings; implemented surfaces pass `bun run build && bun run test:unit` + live E2E.
## Evidence
- `docs/hypotheses.md` §8 (per-H rows); `docs/m365-copilot-api.md` for conclusive protocol facts (source of truth); ticket ## Comments.
## Risks
- Do not drift into native-tool territory (license boundary) — surfacing only.
- The M365 sandbox is not the harness's: egress/FS guarantees absent; keep surfaced code-exec for hashing/math/transforms.
- Thread throttle on verification bursts; ≥3 min spacing, hard stop at first empty-503. Frames may arrive as adaptive cards — dump raw frames when mapping shapes.
