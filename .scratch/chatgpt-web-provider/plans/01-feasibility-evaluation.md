# Plan: Evaluate a ChatGPT-web "free" lane (feasibility, not implementation)
> Ticket: .scratch/chatgpt-web-provider/issues/01-feasibility-evaluation.md · Status: backlog · Blocked by: explicit user opt-in; a free ChatGPT web account (browser-login only, no API key); verify-ephemeral-claims pass on current ToS/limits at eval date

## Purpose
Evaluate `guberm/chatgpt-web-provider` (user-named primary; token-free-gateway,
llm-web-api, web2api kept for comparison) as an optional, opt-in free lane
that hardens the model stack with a second no-cost backend alongside the
free-pool. Decide chat-only vs tool-capable and prove the lane end-to-end
with a real agentic bench task BEFORE wiring anything. Evaluation only — no
implementation commit.

## Preconditions
- User authorization for the eval: creates a real browser session + network
  traffic, and the free account faces ToS/ban risk. The lane MUST stay opt-in
  and NEVER the default route.
- Headed Chromium + Playwright present (M365 interactive login already uses
  them). Browser-driving scripts run under `node` on this PC (playwright
  pipe/connectOverCDP times out under Bun). Python available for guberm.
- Port 8791 free; free-pool LiteLLM (127.0.0.1:8788) or omp custom provider
  reachable for the wire-in step.
- Re-verify free-tier model availability, rate limits, and OpenAI ToS against
  the LIVE source on the eval date (verify-ephemeral-claims skill) — they
  drift.

## Steps
1. Ephemeral-claims check: current ChatGPT free-tier models/limits + ToS
   position on web-session-as-endpoint. Record date + source.
2. Install + configure guberm (MIT, MVP): dedicated ChatGPT browser profile
   (never personal browsing data), model/level allowlist, headed login on the
   PC (same pattern as M365 interactive login), port 8791.
3. Health gate: `/v1/provider/status` (or /health) must show a real
   logged-in session title — NOT "Just a moment..." (Cloudflare). Headless is
   a known dead end; headed on a real desktop session is the proven
   workaround.
4. Chat-completions probe: `/v1/chat/completions` (+ SSE shim), `/v1/models`,
   Bearer auth, serialized concurrency 1, zero/best-effort token usage
   (expected — harnesses will show zeros).
5. Decide the tool-call gap: (a) accept chat-only for planning/chat lanes;
   (b) wrapper injecting the existing fenced-tool framing (packages/core
   fenced + parseToolCalls) and parsing responses; or (c) switch to
   token-free-gateway for prompt-injected tool_calls. Bench the winner, not
   the assumption.
6. Wire-in ONLY if the eval passes: free-pool lane via LiteLLM 127.0.0.1:8788
   (free-pool-ops skill) or omp custom provider (omp-models-yml skill);
   lane health check + fast-fail path so a dead lane never falls through to a
   paid route unintentionally.
7. Bench: `node scripts/bench/run.mjs --base-url <lane>/v1 --model <id>
   --label chatgpt-web-<date> --repeat 5` — n≥5 per task on a real agentic
   task subset; compare pass-rate vs baseline; outputs in scripts/bench/out/.
8. Verdict: `docs/hypotheses.md` — feasible/not as an optional lane, with the
   tool-loop evidence; promote conclusive findings per docs policy; close
   ticket.

## Acceptance
- A real agentic bench task passes end-to-end through the lane (tool loop,
  not just prose) — or the eval fails fast with the recorded reason.
- Lane health visible in the free-pool health check; failure fails fast and
  does not fall through to a paid lane.
- ToS/limits re-verified against the live source on the eval date; verdict
  recorded.
- Nothing defaults to the ChatGPT-web lane (opt-in only).

## Evidence
- `docs/hypotheses.md` (new date-stamped section), `scripts/bench/out/<label>-*.json`,
  ticket `## Comments`. Provider config lives outside this repo (free-pool
  LiteLLM / `~/.omp`) — record endpoints only, no secrets.

## Risks
- ToS/account ban (opt-in only; dedicated profile; never the default route).
- Cloudflare/anti-bot breakage (headed mode; health gate; fast-fail).
- Web-session token expiry + UI-selector fragility (MVP) — the lane needs a
  health check and quick failure, like free-pool lane health.
- Ephemeral model availability/limits (re-verify at eval date).
- Concurrency 1 → fallback lane only, never a primary lane.
- Security: the bridge controls a real Chrome over CDP and holds session
  cookies — dedicated profile, never personal browsing data; no PII or auth
  payloads in the public repo.
