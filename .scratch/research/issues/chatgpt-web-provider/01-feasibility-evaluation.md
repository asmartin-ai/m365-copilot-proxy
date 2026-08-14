# 01 — Evaluate a ChatGPT-web "free" lane for the model stack

**Status:** backlog
**Category:** enhancement
**Type:** research
**Blocked by:** —

## Goal

Evaluate adding a free ChatGPT-web bridge as an optional, opt-in model lane
to harden the stack (a second no-cost backend alongside the free-pool).
If the evaluation passes, wire it as a free-pool lane (LiteLLM
`127.0.0.1:8788`) or an omp custom provider, then bench it.

"chatgpt-web-provider" is an ecosystem term, not one project. It means any
bridge that turns a ChatGPT **web session** (no API key) into an
OpenAI-compatible endpoint. Candidates found 2026-08-10:

| Candidate | Notes |
|---|---|
| `guberm/chatgpt-web-provider` | **User-named candidate (primary).** Python, MIT, MVP. OpenAI-compatible facade over a Playwright persistent Chromium profile driving chatgpt.com. `/v1/chat/completions` (+SSE shim), `/v1/responses`, `/v1/models`, `/v1/provider/status`, Bearer/X-API-Key auth, serialized queue (concurrency 1), model/level allowlist (e.g. `chatgpt-5.6-sol-high-web`, `gpt-5.6-terra`, `gpt-5.6-luna`, `gpt-5.5`, `o3`). **No tool loop yet** (chat-only; README: "no shell/filesystem/MCP tool loop yet"). Cloudflare blocks headless; headed mode on a real desktop session works. Repo self-declares MVP fragility (UI selectors + Cloudflare can change anytime). Port 8791 default |
| `andeya/token-free-gateway` | Alternative. OpenAI-compatible `/v1/chat/completions` + `/v1/models` + streaming + tool_calls (prompt-injected), 13 providers incl. ChatGPT, CDP-controlled Chrome, single binary, daemon mode, Windows build, `webauth` login wizard, creds in `~/.token-free-gateway/auth-profiles.json` |
| `adryfish/llm-web-api` | Classic multi-provider bridge (ChatGPT/Claude/Gemini); verify current state before choosing |
| `ctxinf/web2api-ai-sdk-provider` | SDK-based bridge; newer, less battle-tested |
| OmniRoute "ChatGPT Web" provider | Targets **Plus/Pro** sessions — paid accounts, NOT free. Out of scope |

## Feasibility verdict (2026-08-10)

**Feasible as an optional CHAT lane, with real caveats. The deciding
limitation is tool calling.** Evidence:

- Both leading candidates speak standard OpenAI chat completions, so
  LiteLLM/omp can route to them with zero client changes (same shape as
  the existing free pool).
- Auth is browser-login only — no API key, no card. Credentials stay local
  (dedicated browser profile), matching the project's "no plaintext
  secrets in the repo" rule.
- **guberm has NO tool loop** (chat-only). It cannot drive an agentic
  coding loop as-is. `token-free-gateway` does prompt-injected tool_calls
  (weaker than native — must be benched, not assumed). If guberm wins on
  other axes, tool calls would need a wrapper that injects the fenced-tool
  framing and parses calls — machinery this repo already has in
  `packages/core` (fenced format + parseToolCalls).
- Cloudflare is the main runtime risk: headless Chromium gets stuck on
  "Just a moment..."; headed mode on a real desktop session works. The
  machines already run visible Chromium for M365 interactive login, so the
  headed pattern is proven here (no X11 needed on Windows).
- Runtime deps present: Playwright + Chromium (M365 auth already uses
  them); Python for guberm; node binary for token-free-gateway.
- Concurrency is serialized (1 request at a time) on both — fine for a
  fallback lane, wrong for a primary lane.
- Token usage is zero/best-effort on guberm's browser backend — harnesses
  that display token counts will show zeros.

## Risks (must be stated in any go-ahead)

1. **ToS / account risk**: using a free ChatGPT web account as an API
   endpoint violates OpenAI terms. Accounts can be rate-limited or banned.
   The lane MUST stay opt-in and never the default route.
2. **Ephemeral claims**: free-tier model availability and rate limits
   change. Verify current limits and ToS at implementation time
   (verify-ephemeral-claims skill / deals-watch).
3. **Stability**: web session tokens expire; Cloudflare/anti-bot measures
   can break the bridge; model availability drifts. The lane needs a
   health check and a fast failure path (like the free-pool lane health
   in free-pool-ops).
4. **Security**: the bridge controls a real Chrome over CDP and holds
   session cookies. Run it on the dedicated profile, never with personal
   browsing data.

## Steps when unblocked

1. Evaluate `guberm/chatgpt-web-provider` first (user-named): dedicated
   ChatGPT profile + headed login (same pattern as M365 interactive
   login), allowlist the desired models, confirm `/health` shows a real
   logged-in ChatGPT title (not "Just a moment...").
2. Decide the tool-call gap: chat-only acceptance (planning/chat lanes
   only) vs a wrapper that injects the fenced-tool framing + parses
   responses (reuse `packages/core` fenced machinery) vs switching to
   `token-free-gateway` for its prompt-injected tool_calls. Bench the
   winner, not the assumption.
3. Wire it: add as a LiteLLM provider on the free-pool (free-pool-ops
   skill owns lane wiring) or an omp custom provider (omp-models-yml
   skill).
4. Prove the lane: run `scripts/bench/run.mjs` with a `--label` against a
   real agentic task through the new lane; compare pass-rate vs baseline.
   n≥5 per task.
5. Record the result in `docs/hypotheses.md`; promote a conclusive finding
   per the docs policy.

## Acceptance criteria

- [ ] A real agentic bench task passes through the new lane end-to-end
      (tool loop, not just prose).
- [ ] Lane health visible in the free-pool health check; failure fails
      fast and does not fall through to a paid lane unintentionally.
- [ ] ToS/limits re-verified against the live source on the implementation
      date; verdict recorded.
- [ ] Opt-in only: nothing defaults to the ChatGPT-web lane.

## Comments

- Source: user request 2026-08-10 — "explore potentially trying
  chatgpt-web-provider as an optional 'free' model to harden the system".
- Feasibility research done 2026-08-10 (web + repo READMEs cited above).
- User follow-up 2026-08-10: the specific project is
  `guberm/chatgpt-web-provider` (added as primary candidate above). The
  other candidates (token-free-gateway, llm-web-api, web2api) are KEPT for
  comparison — do not drop them.
- Implementation surface lives outside this repo (free-pool LiteLLM in the
  local LLM stack, omp models.yml in `~/.omp`); this ticket is the backlog
  record and the bench oracle lives here.
