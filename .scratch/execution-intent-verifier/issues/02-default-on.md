# 02 — Flip the verifier default-on

**Status:** resolved
**Category:** enhancement
**Type:** task
**Blocked by:** 01
**Resolution:** implemented 2026-08-08 (default-on with `M365_INTENT_VERIFIER=0`
opt-out; ADR-0002 updated; enacted on architect approval after ticket-01 live
validation cleared the fail-closed/EXECUTE/TEXT paths).

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

---

## Reclassification (2026-08-13 simplify-tool-path)

**Status:** wontfix
**Reason:** Superseded by architecture pivot: the proxy translates observable
M365 output; execution intent/policy belongs to the consuming harness. The
`intent-verifier.ts` / `attestation.ts` modules are preserved as research
artifacts but are no longer on the runtime path (see
`.scratch/simplify-tool-path/spec.md`).
