# Plan: Admin-portal usage dig
> Ticket: .scratch/m365-live-probes/issues/07-admin-portal-dig.md · Status: ready-for-agent · Blocked by: none

## Purpose
Playwright-drive the M365 admin center Copilot usage report and capture the
API call that feeds the dashboard — the last untried route to F5 (a real
metering endpoint). Ecosystem dig (2026-08-11): admin-gated usage exists as
prompt counts (Graph `getMicrosoft365CopilotUsageUserDetail`, needs
`Reports.Read.All` + a limited admin role); unlicensed Copilot Chat is NOT in
Graph. UI-only — zero chat threads.

## Preconditions
- Explicit user authorization for live M365 browser work (standing rule).
- The account must hold an admin role with report access. Check FIRST: no
  admin role → the page renders denied/empty and the dig is dead on this
  account — abort and record the gate (0 threads spent).
- Browser rig per `docs/agents/m365-ui-investigation.md`: persistent profile,
  CDP port 9222, auth via MSAL cache. Drive under `node` (connectOverCDP
  times out under Bun).

## Steps
1. Clear any stale profile lock (`Default/LOCK` in the profile dir), then
   start the browser: `hub start m365-cdp` →
   `node scripts/_profile-cdp.mjs 9222` (runbook sequence).
2. Attach via CDP; navigate to the Copilot usage report under
   admin.microsoft.com (Reports → Usage → Copilot; follow the current nav —
   the URL is session-stamped).
3. Instrument the network: Playwright `page.on('response')` (or CDP Network
   domain) capturing request URL/headers + response body for every XHR while
   the dashboard renders. Identify the call returning the dashboard data
   (Graph `.../copilot/reports/...` or a `reports`-family sibling).
4. Save request + SANITIZED response (strip account identifiers, tenant
   details, PII — public repo) to `experiments/admin-portal-dig/<ts>/capture.json`.
5. Record in `docs/hypotheses.md` §F5: endpoint, auth shape, and what the
   data actually is (prompt counts ≠ tokens; do not claim token metering).
6. Promote to `docs/m365-copilot-api.md` only if the endpoint is confirmed
   useful for the proxy; otherwise leave it as a documented dead end.

## Acceptance
- Captured request/response of the dashboard API (PII stripped).
- Endpoint documented in `docs/hypotheses.md` §F5.
- Promoted to `docs/m365-copilot-api.md` if confirmed useful, with rationale.

## Evidence
- `experiments/admin-portal-dig/<ts>/capture.json`; hypotheses.md §F5 + §7
  row 07; ticket Comments.

## Risks
- Account lacks admin role / license gate → abort at the pre-flight check,
  record the gate (0 threads spent).
- Dashboard data is prompt counts, not token usage (MC1423101) — F5's "no
  self-service token surface" reading likely stands; value is endpoint
  knowledge, not a token oracle.
- Heavy admin pages crash `tab.evaluate`/fullPage — use `_cdp-shot.mjs`
  viewport screenshots + `inspect_image` per runbook.
- Close the browser cleanly after capture (profile-lock footgun); no chat
  threads are involved.
