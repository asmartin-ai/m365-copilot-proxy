# 07 — Leverage M365 built-in tools as proxy features (websearch, images, hosted code execution)

**Status:** backlog
**Type:** research + feature
**Category:** enhancement
**Blocked by:** live M365 (laptop) + rested account; not before tickets 01/02 verdicts

**Source:** user request 2026-08-09 ("as a backlog feature… leverage some of the
m365 built in tools like websearch, images, and its local code execution. That's
a later feature though.")

## Context

M365 Copilot ships real built-in capabilities that the proxy currently underuses
or passes through as text:

1. **Web search** — `plugins:[{Id:"BingWebSearch",Source:"BuiltIn"}]` is already
   sent on the agent-less path (api-doc §4; `nosearchall` disables). Results
   surface as `InternalSearchQuery`/`sourceAttributions` frames; the proxy today
   does not expose them to the harness as a tool or structured citations.
2. **Image generation** — flux flags / `GraphicArt` capability (H8.3, ticket 02);
   image *input* upload flow (H8.10). Genuine `GeneratedCode`/image frames exist
   but are not surfaced to OpenAI-compatible clients.
3. **Hosted code execution** — `cwc_code_interpreter*` optionsSets give a real
   server-side Python sandbox (H8.1, ticket 01, api-doc §5). Verified live with a
   SHA-256 oracle; the proxy enables it agent-less but returns results as plain
   text (observed 2026-08-09: Claude tone ran real Python in M365's sandbox and
   answered truthfully — the proxy passed it through as text).

## Goal

Turn these built-ins into first-class proxy capabilities: decide per tool whether
they surface to the OpenAI-compatible client as a tool, a structured attachment,
or both — e.g. a `web_search` tool call routed to M365's Bing plugin, image
frames mapped to OpenAI image outputs, and the hosted code interpreter as a
sandboxed `execute_python` tool with real execution instead of local-shell
routing.

## Notes / constraints

- **Live toolset evidence (2026-08-09, steering probe):** asked directly, the
  Claude-tone model enumerated its actual M365 tool configuration:
  `search_web` (web search), `web_fetch` (fetch web pages), `python_execution`
  (hosted sandbox), `image_gen` (generate images) — **no shell tool**. This is
  model-level confirmation that M365 ships exactly these built-ins as
  first-class capabilities on the agent-less path; the proxy currently
  intercepts only M365's hosted-shell shape and passes the rest through as text.
  Proxy-side surfacing (not capability acquisition) is the missing piece.
- **Native tool-calling remains OUT OF SCOPE** (AGENTS.md §8.11) — this ticket is
  about surfacing M365's *built-in* capabilities the proxy already invokes, not
  MCP/Dataverse agents.
- Landing order should follow the existing tickets: verdicts from 01
  (code-execution sandbox) and 02 (image canary) first, then this feature ticket
  bundles the survivors.
- The 2026-08-09 observation (Claude tone + hosted Python interpreter engaged
  server-side even with the 5 known flags stripped; proxy returned sandbox answer
  as text) is a live data point for the code-execution half — the sandbox works
  and answers truthfully; the missing piece is proxy-side surfacing, not the
  capability itself.
- Verdicts land in `docs/hypotheses.md`; conclusive protocol facts promote to
  `docs/m365-copilot-api.md`.

## Acceptance (when picked up)

- [ ] Each built-in (search / images / code exec) has a probe verdict in
      `docs/hypotheses.md` with sample size
- [ ] Decision recorded: which surface as tools, which as attachments, which stay
      text passthrough
- [ ] Feature ticket(s) spawned with concrete OpenAI-compatible mappings
