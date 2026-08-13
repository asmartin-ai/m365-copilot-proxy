# Plan: Migrate capture scripts off dead loadSecrets() to msal/persistent profile
> Ticket: .scratch/capture-path-migration/issues/01-migrate-off-loadSecrets.md · Status: needs-triage · Blocked by: none

## Purpose
Finish the auth-path migration the capture scripts missed after the
password→MSAL refactor. The primary script is already migrated; 12 siblings
still import the dead `loadSecrets` export and crash at import time. Goal:
every capture path boots from the msal cache + persistent Playwright profile
with no password/MFA in any script — unblocking GUI frame capture, studio dig,
and sideload tooling for the pi/Codex agent work.

## Preconditions
- `~/.config/opencode-m365/msal-cache.json` + persistent profile
  `browser-profile-cdp` present and logged in (proved live 2026-08-11 via
  `getTokenSilent()`).
- Scripts run under `node` on the PC (playwright connectOverCDP times out
  under Bun 1.3.14 on this box).
- `bun run build` first — scripts import from `packages/core/dist/index.mjs`;
  after build, refresh nested `@m365-copilot/*/dist` copies (stale copies
  shadow workspace links).
- One browser at a time: a second `launchPersistentContext` on the same profile dir = "existing browser session" lock. Interactive re-login needs a human in the open window.

## Steps
1. Verify baseline: `grep -rn "loadSecrets\|loginAutomated" scripts/ da-app/` →
   12 scripts still import `loadSecrets`: `cancel-frame-capture.mjs`,
   `create-full-bot.mjs`, `gateway-capture.mjs`, `gateway-explore.mjs`,
   `login-probe.mjs`, `m365-gui-emulate.mjs`, `studio-dig.mjs`,
   `throttle-recovery-ab.mjs`, `da-app/sideload-and-trigger.mjs`,
   `sideload-devportal.mjs`, `sideload-devportal2.mjs`, `sideload-gui.mjs`.
   `login-probe.mjs` + `throttle-recovery-ab.mjs` additionally import dead
   `loginAutomated` (not exported from core — second crash source).
   (`m365-gui-capture.mjs` is already migrated — use it as the pattern.)
2. Adopt the shipped pattern: `import { getBrowserProfileDir } from
   "../packages/core/dist/index.mjs"`; `chromium.launchPersistentContext(
   getBrowserProfileDir(), { headless: false, timeout: 60_000, ...
   (CHROMIUM_PATH override), args: ["--no-sandbox","--disable-dev-shm-usage"] })`;
   login fallback = detect `/login\.microsoftonline/` then wait for the user's
   interactive sign-in in the OPEN window (waitForURL off the login tenant) —
   never a second context, never password/MFA/TOTP filling.
3. Migrate each sibling: delete the `loadSecrets()` call + "no secrets" exit
   paths + the loginfmt/passwd/otc fill helper (and its TOTP import); swap in
   the persistent-profile boot; keep each script's capture purpose (WS frame
   capture, optionsSets/agent dig, app sideload, cancel-frame capture).
4. `login-probe.mjs` + `throttle-recovery-ab.mjs`: also drop `loginAutomated`; use `getTokenSilent()` for the silent path; for a fresh-login token path use `loginInteractive()` from `packages/core/src/auth.ts` (nativeclient redirect via `extractAuthCode` — no password in the script).
5. `grep loadSecrets scripts/ da-app/` → zero hits; `grep loginAutomated
   scripts/ da-app/` → zero hits.
6. Smoke: `node scripts/m365-gui-capture.mjs "task"` plus one migrated sibling
   (e.g. `gateway-capture.mjs`) — boots a logged-in persistent browser, captures
   frames, no interactive login needed. Sequential; these are GUI captures, not
   chat turns — keep M365 work thread-conserving regardless.
7. Update `docs/m365-copilot-api.md` auth section if it still documents the
   secrets.json flow; `docs/agents/m365-ui-investigation.md` already lists the
   migrated helpers.

## Acceptance
- `grep loadSecrets scripts/ da-app/` → zero hits; `grep loginAutomated` →
  zero hits.
- `m365-gui-capture.mjs` and ≥1 sibling boot a logged-in persistent browser and
  capture WS frames with no password/MFA in the script and no secrets.json
  dependency.
- Auth docs updated to the msal/persistent-profile path.

## Evidence
- Ticket ## Comments — per-script migration log + smoke output (frame counts).
- `docs/m365-copilot-api.md` auth section — old flow removed/rewritten.
- `docs/agents/m365-ui-investigation.md` — already reflects the migrated
  helpers (`_profile-cdp.mjs`, `_cdp-shot.mjs`, `_profile-diag.mjs`).

## Risks
- Dist staleness: stale nested `@m365-copilot/*/dist` copies shadow workspace
  links — rebuild + refresh before smoke.
- Profile lock: one launchPersistentContext per profile dir at a time; any
  signed-out profile needs a human interactive sign-in.
- Keep all M365 work sequential and thread-conserving; any script that sends
  chat turns counts against the probe budget (≤12/hr, ≥3 min spacing, hard
  stop at first empty-503/at-limit).
