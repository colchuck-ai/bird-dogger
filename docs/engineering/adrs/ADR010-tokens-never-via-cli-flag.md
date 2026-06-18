# ADR010 - Tokens Never via CLI Flag

Source tokens and secrets are never accepted as CLI flags. The only
valid input mechanisms are an interactive terminal prompt (for
`source add` and `source rotate-token`) and the
`BDOG_SOURCE_<NAME>_TOKEN` environment-variable fallback for headless
environments. Flag shapes that carry a secret as an argument — `--token`,
`--clear-token`, or any equivalent — are architecturally foreclosed.
This constraint applies at C003 Credential Store and C005 Source
Adapter, not just at the CLI verb surface.

## Context

`source add` and `source rotate-token` are the two CLI paths that write
source credentials into C003's keyring store. Both need an input
mechanism for the secret string. The question is which mechanisms are
acceptable.

Any value passed as a CLI flag appears in two persistent records by
default:

- **Shell history.** Most shells record the full command line, including
  flag values, in a persistent history file. On a developer workstation
  this file is readable by any process running as the same user; on
  machines where the account is shared (CI agents, shared developer
  hosts), the exposure is wider.
- **Process list.** While the command is running, `ps aux` and
  `/proc/$PID/cmdline` expose the full argument vector to any process
  with the right permissions. On many operating systems this includes all
  local users without privilege escalation.

An interactive terminal prompt reads the secret from the terminal's line
discipline without writing it to the shell history file or the process
argument vector. The `BDOG_SOURCE_<NAME>_TOKEN` env-var — where `<NAME>`
is the source name uppercased with `-` replaced by `_` (e.g., `ttd-jira`
→ `BDOG_SOURCE_TTD_JIRA_TOKEN`) — provides the headless path for CI and
scripted environments. Environment variables appear in
`/proc/$PID/environ` rather than the argument vector; they are not
recorded in shell history unless the user explicitly writes them into a
history-recording script.

A `--token` flag appears superficially convenient for scripting and
onboarding. The convenience is real; the exposure is also real and
silent — the bird-dogger may not realize their token is now in
`~/.zsh_history`. Without a recorded decision, a `--token` flag could
appear as a "CI convenience" addition in future development without
surface-level awareness of the permanent exposure.

`--clear-token` is a related flag shape that would express "remove this
source's token from the keyring" without removing the source declaration.
This shape presupposes a valid state where a source exists without a
credential. That state has no useful role: a source with no valid token
cannot be used and has no meaningful declaration to preserve. Credential
removal happens via `source remove`, which removes the source declaration
and its credential together.

## Options

- **A. Interactive prompt and env-var only; no flag.**  
  `source add` (for token-bearing types) and `source rotate-token`
  always prompt interactively when stdin is a terminal.
  `BDOG_SOURCE_<NAME>_TOKEN` provides the headless fallback.
  No `--token` flag. No `--clear-token` flag. Cost: the bird-dogger
  cannot inline the secret in a shell one-liner; CI environments must
  inject credentials via env-vars.

- **B. Accept `--token <value>` on `source add` and `source rotate-token`.**  
  Token accepted as a flag value alongside the interactive-prompt path.
  Cost: the token appears in shell history on every invocation that uses
  the flag; it appears in the process list while the command runs; the
  exposure is permanent and the bird-dogger has no in-tool warning.

- **C. Accept `--token` with documented risk warning.**  
  Same as option B, augmented with help-text and documentation warnings
  about shell history exposure. Cost: documentation does not prevent
  use; a warning in help text creates a false safety signal (the option
  is listed and documented, therefore presumably vetted) without removing
  the exposure; the history record is permanent even if the bird-dogger
  reads the warning afterward.

## Decision

Option A — interactive prompt and env-var fallback; no CLI flag.

The convenience of `--token` is not worth the exposure. Shell history
and process lists are not adversarial surfaces in every environment, but
the bird-dogger has no reliable control over them. Using the flag
requires every bird-dogger to understand the risk, take deliberate
precautions (`HISTIGNORE`, shell-history suppression, process-list
filtering), and verify those precautions worked. The interactive-prompt
path requires no precautions: the terminal line discipline handles the
secret and the shell records only the command name, not the input.

The env-var path meets the headless requirement without shell-history
exposure. The Engineering README's Constraints section already commits to
the keyring-and-env-var model as the complete secret storage shape; this
ADR records why the flag shape was rejected rather than left open.

`--clear-token` is foreclosed for the same structural reason: a source
without a valid credential is not a partial success state worth
preserving. Removing a source's credential is inseparable from removing
the source declaration that binds to it.

This constraint is architectural, not just CLI style. It governs C003
Credential Store — which must never accept a token string delivered from
a flag value — and C005 Source Adapter — which receives credentials
exclusively from C003, not from any CLI-layer input. A future verb or
flag that carried a secret as an argument would violate C003's write-path
contract regardless of how the verb was named.

## Consequences

- `source add` (for token-bearing types) prompts interactively when
  stdin is a terminal. `BDOG_SOURCE_<NAME>_TOKEN` is the headless path.
  No `--token` flag exists on any command.
- `source rotate-token` follows the same pattern: interactive prompt
  and env-var fallback; no `--token` flag. See
  [ADR009](ADR009-side-effecting-actions-on-dedicated-verbs.md) for why
  `rotate-token` is a dedicated action verb rather than a flag on
  `source set`.
- No `--clear-token` flag exists on any verb. A source's credential is
  removed by removing the source declaration (`source remove`). The
  no-cascade-removal rule (see
  [ADR007](ADR007-no-cascade-removal.md)) ensures that a source with
  active consumers cannot be silently removed.
- C003 Credential Store's write paths are `source add` (new token) and
  `source rotate-token` (replacement). Both originate from interactive
  prompt or env-var reads. No CLI-layer code delivers a token string to
  C003 from a flag value.
- C005 Source Adapter receives credentials exclusively from C003 at
  request time. C005 has no token-accept interface of its own; an
  adapter implementation that accepted a token directly would violate
  this constraint.
- The `BDOG_SOURCE_<NAME>_TOKEN` naming scheme is load-bearing:
  `<NAME>` is the source name uppercased with `-` replaced by `_`.
  CLI help text for `source add` and `source rotate-token` must describe
  the env-var path alongside the interactive prompt.
- Future source types that carry credentials follow the same model:
  interactive prompt on `source add` and `source rotate-token`, env-var
  fallback. Tokenless types (e.g., `local-json` per
  [ADR005](ADR005-local-json-source-type.md)) skip both paths; C003 is
  not asked.
- Composes with
  [ADR009](ADR009-side-effecting-actions-on-dedicated-verbs.md):
  credential operations (`source add`, `source rotate-token`) are
  action verbs with distinct idempotency postures; neither is a flag on
  `source set`.
- Composes with
  [ADR005](ADR005-local-json-source-type.md): `local-json` sources
  carry no secret; the interactive-prompt and env-var paths are bypassed
  entirely for tokenless source types.
