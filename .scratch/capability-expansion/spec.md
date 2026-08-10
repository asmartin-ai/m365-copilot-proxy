# M365 Capability Expansion

**Status:** Backlog — probes need a live M365 (laptop); `optionsSets` is
currently sent empty.
**Source:** `docs/hypotheses.md` §8 (June 13 web-research dig)

## Why this matters

The chat payload sends `optionsSets: []` — empty. Every other implementation
(Microsoft red-team tool, PyRIT, kuchris, g365, SydneyQt) ships rich
`optionsSets` arrays. We are likely leaving capabilities off the table by
omission; several of those capabilities (real code execution, native tool
calls, memory) attack the live prose-compliance problem from the capability
layer, not the prompt layer.

## Hard constraint

**Native tool-calling is permanently OUT OF SCOPE** (AGENTS.md §8.11, backed
into CONTEXT.md): MCP or a full Dataverse bot would need a Copilot Studio
license, breaking the zero-cost premise. Tickets 03 (H8.4/H8.5) exist as a
hostile-record of that boundary, not as work to start.

## Ticket map

| # | Scope | Hypotheses | Blocked by |
|---|-------|-----------|------------|
| 01 | Real server-side code-execution sandbox | H8.1, H8.2 | laptop + rested account |
| 02 | Image generation via capabilities | H8.3 | laptop |
| 03 | Native actions/plugins (MCP) | H8.4, H8.5 | **OUT OF SCOPE** |
| 04 | Model selection & smart-mode routing | H8.6, H8.7, H8.8 | laptop |
| 05 | Grounding & multimodal | H8.9, H8.10, H8.11, H8.12 | laptop |
| 06 | Memory, instructions, behavior | H8.13, H8.14, H8.15, H8.16 | laptop |

Backlog feature (user 2026-08-09): **07 — surface M365 built-in tools to
OpenAI-compatible clients** (web search / images / hosted code execution) as
tools or structured output. See `issues/07-m365-builtin-tools-feature.md`.

Verdicts **always** land in `docs/hypotheses.md` (sample size + evidence)
and get promoted to `docs/m365-copilot-api.md` when conclusive — that file
is the source of truth, not the tickets.