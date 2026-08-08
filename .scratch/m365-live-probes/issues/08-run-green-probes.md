# 08 — Run the green probes

**Blocked by:** laptop + rested account
**Type:** research
**Category:** enhancement

## Context

The five 🟢 probes have scripts already; they haven't been run on a rested
account. Run them now the account is fresh:

- `scripts/usage-endpoint-hunt.mjs` (F5, 0 msgs)
- `scripts/variants-bisect.mjs` (~10 msgs/target — also feeds ticket 05)
- `scripts/frame-dump-probe.mjs` (1 msg — catch newly-added fields)
- `scripts/frame-dump-disengage.mjs` (1 msg — F6)
- `scripts/tool-compliance-experiment.mjs --repeat N` (30N msgs — see 02)

## Acceptance

- [ ] Each probe run on a rested account; one variable per run
- [ ] Results logged in `docs/hypotheses.md` with sample size
- [ ] Conclusive findings promoted to `docs/m365-copilot-api.md`