# 03 — Wontfix survey: obsolete scratch tickets

**What to build:** A pass over every `.scratch/<feature>/issues/` ticket (35+ files) and the top-level `CONTINUATION_PROMPT.md`, classifying each against the new boundary:

- **Product** — keep/rewrite (protocol/transport/IP).
- **Research** — move the ticket into `.scratch/research/issues/`.
- **Contradicts the pivot** — mark `wontfix` with reason: "Superseded by architecture pivot: the proxy translates observable M365 output; execution intent/policy belongs to the consuming harness."
- **Uncertain** — leave open with a `needs-triage` tag, but do not keep it active.

Boundary rule (GPT-4o's):
- Product tells M365 how to represent the client's request → keep.
- Product tells M365 what decision to make on the client's behalf → wontfix/research.

Apply the keep/delete rule to unit tests as part of this sweep (the 8H verifier, attestation, confab classifiers, read-only fallback = research artifacts, not product).

**Blocked by:** 01 (so the boundary is real before we reclassify).

**Status:** ready-for-agent

- [x] All `.scratch/*/issues/*.md` files classified (40 product-keep tickets in-place; 21 research tickets relocated to `.scratch/research/issues/`; 9 contradicting tickets marked wontfix).
- [x] Wontfix tickets carry the standard reason line ("Superseded by architecture pivot…").
- [x] Research tickets relocated under `.scratch/research/issues/`.
- [x] The 8H / attestation / injection-ladder tickets reclassified (not deleted) — verifier + attestation tickets marked wontfix; steering-gate tickets (#02, #03) kept as Product (steeringFingerprint survives the pivot); probe results moved to research.
- [x] `git ls-files .scratch/*/issues` shows the new layout (see checkout below).
- [x] Doc-sync pass complete: README, ARCHITECTURE_ROADMAP, COMPONENT_REFERENCE, tool-calling.md, ADR-0002, hypotheses.md §9/§14, CONTINUATION_PROMPT.md updated.
