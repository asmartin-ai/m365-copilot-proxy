# Client-attested execution

**Status:** implemented, opt-in (2026-08-09)

## Goal

Offer an explicit, opt-in path where pi, Oh My Pi, or Codex gives the final
permission for one exact `bash` tool call. This path bypasses the local 8H
execution-intent classifier latency only for a configured client adapter that
blocks execution until the proxy accepts its attestation.

The existing 8H verifier remains the default for every request that does not
opt in. This feature does not change M365 protocol framing, tool parsing, or
ordinary OpenAI-compatible behavior.

## Boundary

The proxy emits tool calls. The client harness executes them. The proxy cannot
make the client execute a command, and it cannot observe a native approval
prompt. The adapter hook is the execution boundary.

The adapter must:

1. Receive the proxy-issued `tool_call_id` and the exact command that its shell
   tool will execute.
2. Apply its normal policy and user approval UI. DCG remains an Oh My Pi deny
   floor.
3. Ask the loopback proxy to consume a one-time authorization for that exact
   command.
4. Execute only after the proxy returns `allow`.

A client without this adapter must not opt in.

## Request selection

A request selects this path only when all conditions hold:

- `M365_CLIENT_ATTESTATION=1` is set on the proxy.
- The request has `X-M365-Execution-Gate: attestation-v1`.
- The request has `X-M365-Attestation-Client: pi`, `omp`, or `codex`.
- The request has `X-M365-Attestation-Proof` =
  HMAC-SHA256(secret, `"attestation-v1\n" + client`) hex — proves the caller
  holds the shared secret. A bare header must not strip the 8H gate.
- The configured shared secret for that client exists.

Any missing or invalid condition uses the existing 8H verifier path. There is
no permissive fallback.

## Candidate lifecycle

After parsing and normalizing a single `bash` call, but before the proxy emits
it, the proxy creates an in-memory candidate:

- `tool_call_id`: the proxy-issued OpenAI `call_…` identifier.
- `client`: the selected client identifier.
- `command_sha256`: SHA-256 of the exact `function.arguments.command` string
  emitted to the client. This binds Windows shell wrapping and other parser
  normalization, not the source fence.
- `expires_at`: 60 seconds after creation.
- state: `PENDING`, `AUTHORIZED`, or `RESULT_ACCEPTED`.

The proxy atomically changes a matching `PENDING` candidate to `AUTHORIZED`
when it returns `allow`. This consumes the approval: a second attestation
cannot authorize the same candidate. A tool execution failure still consumes
the approval.

The next tool-result request must reference an unexpired `AUTHORIZED`
candidate. The proxy changes that candidate to `RESULT_ACCEPTED` before it
continues the M365 turn. A missing authorization denies the result and stops
the agent loop. This does not undo a client execution; the hook remains the
pre-execution boundary.

Process restart, eviction, expiry, an unknown id, or any mismatch deny
execution. Version 1 accepts no id-less fallback. It does not infer a candidate
from a command hash or a tool result. The one exception: a tool-result id the
proxy itself emitted through the 8H verifier path (tracked by `SessionPool`)
is accepted without a candidate — that path never registers one. An id that
was never emitted by this proxy is always denied (fabricated ids cannot
bypass the gate). Validation covers every tool message in the request before
any approval is consumed, so a 409 does not burn earlier candidates.

## Attestation endpoint

`POST /v1/attestations` is a loopback-only control endpoint. It rejects a
forwarded address. The initial payload is:

```json
{
  "client": "pi",
  "tool": "bash",
  "tool_call_id": "call_…",
  "command_sha256": "lowercase SHA-256 hex",
  "ts": 1786263011,
  "nonce": "base64url-random"
}
```

The hook sends `X-M365-Attestation-Sig` as lowercase hex
`HMAC-SHA256(secret, client + "\n" + tool + "\n" + tool_call_id + "\n" + command_sha256 + "\n" + ts + "\n" + nonce)`.

The endpoint checks all of these before it changes a candidate to `AUTHORIZED`:

- Client and tool are valid.
- The timestamp is within 60 seconds.
- The nonce has not appeared in the active window.
- The signature is valid in constant time.
- Candidate client, id, command digest, state, and expiry all match.

Success returns `{ "decision": "allow" }`. Every failure returns a denial;
the hook blocks the tool and does not run it.

## Security model

The shared secret authenticates a configured adapter to the local proxy. It is
not a sandbox against code that the user already permits to run as the same OS
user. The feature shifts permission from the probabilistic 8H classifier to a
trusted local harness only when the user explicitly configures that harness.

A model cannot select this route by writing a fence: only the client sends the
initial provider request header, and only the hook can complete the signed
attestation. The adapter still remains responsible for blocking tool execution
when the endpoint denies it.

## Wire compatibility

- Chat Completions: the emitted `tool_calls[].id` is the candidate id.
- Responses: the emitted `function_call.call_id` is the same candidate id;
  `fc_…` item ids are not authorization identifiers.
- The next tool-result request must reference that candidate in `AUTHORIZED`
  state. The proxy changes it to `RESULT_ACCEPTED` before accepting the result.
  This closes the loop but does not claim to prove client execution.

## Non-goals

- Direct proxy-side shell execution.
- Proving that a human approved a command.
- Multiple commands per turn.
- Support for an adapter that strips the proxy tool-call id.
- Changing the frozen 8H corpus, prompt, or held-out evaluation.
- M365 live testing for this control-plane-only work.

## Evidence

- `docs/research/client-approval-attestation.md`
- `docs/adr/ADR-0002-EXECUTION-INTENT-VERIFIER.md`
- `docs/m365-copilot-api.md` §8.10
