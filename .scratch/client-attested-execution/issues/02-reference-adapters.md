# 02 — Reference adapters for pi, Oh My Pi, and Codex

**Status:** resolved (2026-08-09)
**Category:** enhancement
**Type:** task
**Blocked by:** 01
**Spec:** ../spec.md

## What to build

Ship small, copyable reference adapters for the client-attestation contract.
They use one zero-dependency ESM helper and three thin client wrappers. The
helper must be portable: it has no `@m365-copilot/*` imports and runs directly
with Node or Bun.

## Required files

```text
client-adapters/
  attestation-helper.mjs
  pi-attestation-gate.ts
  omp-attestation-gate.ts
  codex-hooks.json
  README.md
```

The helper receives the exact client-visible command and proxy-issued tool-call
id, calculates its SHA-256 digest, creates a nonce and timestamp, signs the
specified canonical string with the configured HMAC key, and posts to the local
proxy. Any parsing, configuration, network, timeout, or proxy failure blocks
the tool. It must never execute a command.

The two TypeScript wrappers only adapt their event data to the helper:

- pi: `pi.on("tool_call", …)`; install via `pi --extension <path>` or the pi
  extension directory.
- Oh My Pi: `.omp/hooks/pre/*.ts` or `--hook <path>`; DCG remains active.

`codex-hooks.json` uses the current event-keyed `PreToolUse` format with matcher
`Bash` and invokes the helper as a command hook. Its command output blocks on a
denial and approves only after a proxy allow response. The README must state
that Codex hook trust review is required and that `PreToolUse` is the reliable
gate even when the permission mode does not request an interactive prompt.

## Acceptance criteria

- [x] The helper accepts the pi/OMP event form and the Codex stdin JSON form,
  preserving `tool_call_id` and the exact `command` string without normalization.
- [x] The HMAC and JSON payload exactly match `.scratch/client-attested-execution/spec.md`.
- [x] A non-2xx response, malformed input, missing configuration, or timeout
  blocks tool execution in every wrapper.
- [x] The Codex `PreToolUse` response uses the current `"approve" | "block"`
  top-level decision values. Do not ship the old versioned hooks-array format.
- [x] The pi and OMP wrappers return `{ block: true, reason }` on denial.
- [x] The helper uses only `node:crypto`, web `fetch`, and standard ESM.
- [x] Tests cover signing, a successful allow, each denial mode, and both input
  adapters. No harness binary, M365 request, or real command execution runs.
- [x] README install steps identify the proxy URL, client id, secret location,
  hook loading command/path, and the fact that a user or local harness policy
  remains the execution authority.

## Source evidence

- `docs/research/client-approval-attestation.md`
- pi: `packages/coding-agent/examples/extensions/permission-gate.ts`
- OMP: `omp://hooks.md`
- Codex: https://learn.chatgpt.com/docs/hooks;
  `codex-rs/hooks/src/schema.rs` (`PreToolUseDecisionWire`)

## Out of scope

- Replacing client approval UI or DCG.
- Storing the HMAC key in a model-visible prompt or tool payload.
- Direct proxy-side command execution.

## Comments

- Resolved 2026-08-09: `bun run test` passed (254 passed, 3 live-gated skipped).
  Adapter-focused coverage passed: 6 tests for canonical signing, deny paths,
  Codex input, and pi/OMP hook behavior. No M365 request was made.
