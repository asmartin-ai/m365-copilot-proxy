# 03 — Native plugin tools (MCP / OpenAPI actions)

**Status:** wontfix
**Category:** enhancement
**Type:** research (recorded only)
**Blocked by:** license (indisputable)
**Source:** `docs/hypotheses.md` §8.1 H8.4, H8.5

## Why this ticket exists

H8.4 (embedded `ai-plugin.json` with OpenAPI runtime) and H8.5 (RemoteMCPServer runtime) promise **real native tool execution** — Copilot calling our tools instead of us emulating them. They are the documented holy grail.

**But they are permanently OUT OF SCOPE.** Making them work needs a Copilot Studio license (MCP / Dataverse bot), which breaks the zero-cost premise. AGENTS.md — **Native tool-calling is permanently OUT OF SCOPE**, backed into `CONTEXT.md`.

The AGENTS.md decision stands unless a human reverses it. Do NOT start this ticket; do NOT delete it — it is the historic record for the "why didn't you build native tools" question that will recur.

**Reverse the hold:** a license is available or price math changes → delete
this sentence, set `Status: ready-for-agent`, and re-plan H8.4/H8.5 as a
real ticket.