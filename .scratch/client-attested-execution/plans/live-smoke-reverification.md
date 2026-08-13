# Plan: Live smoke — attestation loop re-verification with proof header
> Ticket: NEXT.md "Next session plan" item 3 (no ticket file) · Status: pending · Blocked by: explicit user authorization for live M365 probes; M365 auth state (msal-cache.json)

## Purpose
Prove the opt-in client-attested execution loop end to end against live M365:
register → AUTHORIZED → tool result accepted through a real harness (pi / OMP /
Codex), and re-verify live the `X-M365-Attestation-Proof` change the 2026-08-09
laptop smoke predates (proof now REQUIRED to opt in; bare headers must keep the
8H path). Unblocks the latency win (8H verifier skipped for one attested bash
call) without touching the frozen 8H fail-closed baseline. Uses the existing
prompt-emulated bash tool; native tool-calling stays out of scope.

## Preconditions
- Explicit user authorization for live M365 probes (hard rule: standby-only until given). Strictly sequential, one thread at a time.
- Repo on rewritten `origin/main`, clean tree. `bun run build && bun run test:unit` green (never bare `bun test`); after build refresh nested `packages/proxy/node_modules/@m365-copilot/*/dist` copies.
- LM Studio serving `bonsai-27b`; proxy runs with `M365_INTENT_VERIFIER_MODEL=bonsai-27b` (identity guard rejects the default id). Needed for negative A where the 8H verifier must actually run.
- M365 auth present (`msal-cache.json`; interactive login done on PC 2026-08-10 — docs/agents/m365-auth-workflow.md). Laptop: `git fetch origin`, rebase onto rewritten main before live work.
- Proxy env: `M365_CLIENT_ATTESTATION=1`, `M365_ATTESTATION_SECRET=<random>`, `M365_ATTESTATION_URL=http://127.0.0.1:<proxy-port>`. Helper env: same secret + URL. No other probe/proxy session running.

## Steps
1. Generate proof headers (once per client): `bun client-adapters/attestation-helper.mjs --proof pi|omp|codex`.
2. Wire one harness per `client-adapters/README.md` §2–3:
   - pi: gate headers in `~/.pi/agent/models.json` + `pi --extension <abs>/pi-attestation-gate.ts`.
   - OMP: headers in `~/.omp/agent/models.yml` + `omp --hook <abs>/omp-attestation-gate.ts` (DCG stays as deny floor).
   - Codex: headers in `~/.codex/config.toml` + `PreToolUse` Bash entry from `codex-hooks.json` into `~/.codex/hooks.json` (hook trust review per README).
3. Start the proxy; sanity-check the control route: loopback `POST /v1/attestations` returns 404 when gate env is missing, is reachable (non-404) with env set, and rejects a forwarded address with 404.
4. POSITIVE (1 conversation = 1 thread): prompt one bash call, e.g. "run echo hello". Approve the hook's confirm UI → helper signs and posts → expect 200 `{"decision":"allow"}` → shell executes → tool-result request carries the same 3 headers → 200 and the M365 turn continues. With `M365_DEBUG=1` confirm candidate PENDING→AUTHORIZED→RESULT_ACCEPTED and that the 8H verifier did NOT run for that turn.
5. NEGATIVE A — no proof (1 conversation = 1 thread): repeat with `X-M365-Attestation-Proof` omitted or wrong → gate headers ignored, request stays on the 8H path. Debug trace must show the LM Studio verifier ran (EXECUTE check) and no candidate registered; tool result still accepted via the SessionPool-emitted id.
6. NEGATIVE B — fabricated id (0 threads; rejected before any M365 traffic): send a tool-result request with the selection headers but a never-emitted id (`call_fake_…`) → expect 409 `attestation_required`, no M365 traffic.
7. Discipline: ≥3 min spacing between conversations; hard stop at the first empty-503/at-limit. An empty reply WITHOUT a Disengaged frame is a thread throttle, not a content filter.

## Acceptance
- Positive: 200 `allow` from `/v1/attestations`; harness executed the command; tool result accepted; M365 reply received; debug trace proves the 8H verifier was skipped for that turn.
- Negative A: gate headers without a valid proof keep the 8H path (verifier ran; no candidate registered).
- Negative B: 409 `attestation_required` on a never-emitted id, before any M365 traffic.
- Budget respected: ≤3 conversations total, ≥3 min spacing, no 503 hit (or clean hard stop at the first one).

## Evidence
- Verdict + trace excerpts in `.scratch/client-attested-execution/issues/01-attestation-gate.md` ## Comments; tick NEXT.md next-session item 3 on pass.
- A behavior discrepancy (not just a smoke failure) → note in `docs/hypotheses.md` before any code change. No `experiments/` run needed for a control-plane smoke.

## Risks
- 60 s TTL: approval must complete within 60 s of candidate emission — pre-stage secret/headers and approve immediately.
- Thread budget: ≤12 fresh conversations/hr, ≥3 min spacing, hard stop at first empty-503/at-limit; empty reply without Disengaged frame = throttle.
- n=1 noise: this is a smoke, not an experiment — no `--repeat`; assert on tool_call presence/framing, not reply wording.
- Fail-closed: any env/header/hook mismatch must fall back to 8H or deny (403/409) — never a permissive path.
- Secret hygiene: the shared secret must not appear in prompts, tool args, logs, or committed files (public repo, no PII).
- Process restart wipes the in-memory registry — candidates are per-run; restart invalidates in-flight smoke state.
- LM Studio identity guard: confirm `bonsai-27b` is loaded before negative A so the verifier truly runs.
