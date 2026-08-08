# 02 — Flip the verifier default-on

**Status:** ready-for-agent
**Category:** enhancement
**Type:** task
**Blocked by:** 01

## Goal

Once live validation (01) proves parity and no throttle interaction, flip
the verifier gate to default ON so tool execution is fail-closed without an
env flag. This is the "separately approved production integration" step the
10A decision rule requires.

## Acceptance

- [ ] `M365_INTENT_VERIFIER=1` no longer required for the gate to engage
- [ ] Default ON keeps the fail-closed arbitration (only EXECUTE executes)
- [ ] Existing deployments can opt OUT with an explicit env switch
- [ ] Docs (`docs/adr/ADR-0002`, `docs/m365-copilot-api.md` if needed)
      updated to the new default

**Out of scope:** held-out evaluation; corpus/prompt changes (frozen).