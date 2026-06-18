# ADR009 - Side-Effecting Actions on Dedicated Verbs

Side-effecting operations — recording events, advancing state machines,
and producing new timestamps — live on dedicated action verbs, not as
flags or sub-commands on `set`. The `add|remove|info|list|set` pattern
governs field-oriented sub-commands but is not the complete verb
inventory; action verbs are architecturally distinct.

## Context

The CLI's noun-first command pattern organizes sub-commands under a
standard verb layer: `add`, `remove`, `info`, `list`, and `set`. Each
verb has an unambiguous role: `add` creates, `remove` deletes,
`info`/`list` read, `set` writes named fields.

Several operations fit none of those shapes. `source rotate-token`,
`hunt activate`, `hunt deactivate`, `item accept`, `item reject`,
`item carry-forward`, `item confirm-owner`, and `touch record-response`
are not field writes — they record an event, flip a state machine,
or advance a timestamp. The idempotency posture is different too: `item
accept` errors with `already in active set` on re-invocation; `item
confirm-owner` is explicitly not idempotent (each call advances the
confirmation timestamp). Neither behavior fits `set`, which is
idempotent by construction (writing the same field value twice is a
no-op).

Without a recorded decision, these operations gravitate toward `set`
under one of two surface patterns:

- **Valueless flag on `set`** — `bdog item set <item> --accept`. The
  flag carries no value; its presence triggers the action. The parsing
  model accepts `--field value` and `--flag` as separate forms, but
  mixing them under `set` blurs the boundary between "change this
  field" and "trigger this action."
- **Value-bearing flag on `set`** — `bdog item set <item> --status
  active`. Looks like a field write but actually drives a state
  machine. `set` has no way to distinguish "write the literal string
  `active`" from "execute the activation transition," and so it cannot
  report state-machine errors that don't apply to field writes (e.g.,
  `already active`).

Both patterns eventually require `set` to carry state-machine and
event-recording logic it was not designed to hold. A third option —
a generic action namespace like `bdog item do accept` — avoids the
`set` contamination but adds a structurally meaningless dispatch layer
that obscures what the operation actually does.

## Options

- **A. Dedicated action verbs.** Each side-effecting operation has its
  own verb at the noun's command surface: `item accept`, `item reject`,
  `item confirm-owner`, `hunt activate`, `hunt deactivate`, `source
  rotate-token`, `touch record-response`. The verb is the action; no
  flag plumbing needed. Each verb owns its error surface
  (`item accept` → `already in active set`; `hunt activate` →
  `already active`, no-op). Cost: the `add|remove|info|list|set`
  template is not the complete verb list; new side-effecting operations
  demand new verbs rather than a new flag on `set`.
- **B. Valueless flags on `set`.** Encode actions as boolean flags:
  `bdog item set <item> --accept`, `bdog hunt set <hunt>
  --activate`. Cost: `set` must now carry action dispatch logic; flag
  and field writes share a parse path; the valueless-flag anti-pattern
  (`--accept` has no argument, unlike every other `--field value` flag
  on `set`) is inconsistent; error reporting for action failures
  (`already in active set`) has no natural home in a field-update verb.
- **C. Value-bearing field simulation on `set`.** Expose actions as
  apparent field writes: `bdog item set <item> --status active`.
  Looks uniform but encodes semantics in the value string rather than
  in the verb. Cost: `set` must validate legal state transitions for
  what appears to be a field; the error vocabulary (`already in active
  set`, `cannot transition from rejected`) bleeds into field-update
  territory; "fields" that are actually state machines are
  indistinguishable from plain fields in help text and tab completion.
- **D. Generic action namespace.** Route all actions through a `do`
  sub-command: `bdog item do accept`, `bdog hunt do activate`. Cost:
  `do` is a meaningless dispatch wrapper that adds a required word
  without describing the operation; it solves the `set` contamination
  but at the cost of a noun-verb-action triple where the middle word
  contributes nothing.

## Decision

Option A — dedicated action verbs.

The verb layer under each noun is not a closed set constrained to
`add|remove|info|list|set`. Those five cover the field-management
shape; side-effecting operations need their own names because they
*are* their own operations. `item accept` is not a variation of
`item set`; it is a distinct action with a distinct precondition, a
distinct error vocabulary, and a distinct idempotency posture.
Naming it as such is more honest than routing it through a field-update
verb.

The valueless-flag and value-simulation alternatives (B and C) both
place the architectural boundary at the wrong level: they let `set`'s
surface look uniform while pushing state-machine complexity into the
implementation layer below it. The failure surface — what errors can
the bird-dogger see, and what do they mean? — is also degraded: `bdog
item set <item> --accept` returning `already in active set` is
confusing because `set` has no other concept of "already done" states.
`bdog item accept` returning the same message is self-explanatory.

The generic-action-namespace option (D) solves the `set` contamination
but introduces structural noise; the middle `do` word adds nothing.

## Consequences

- `add|remove|info|list|set` is not the exhaustive sub-command
  template for every noun. Each noun's verb list is the union of the
  field-management verbs that apply to it and any action verbs specific
  to its behavior. CLI documentation must list action verbs alongside
  field verbs without implying the two classes are equivalent.
- Every new side-effecting operation requires a dedicated verb, not a
  new flag on `set`. The test: "is this a field write or a
  state transition / event record / timestamp advance?" If the latter,
  it gets its own verb.
- `set` retains a narrow, idempotent contract: write named fields;
  return the updated record. No state-machine logic. No event
  recording. No timestamp advancing. A future `set` sub-command that
  violates this contract is a signal that the operation should be a
  dedicated verb instead.
- Error reporting is precise per action verb. `item accept` errors on
  `already in active set`; `item reject` errors on `already rejected`;
  `hunt activate` succeeds as a no-op if the hunt is already active
  (idempotent). These semantics are unreachable from `set` without
  special-casing that would break its uniform contract.
- The idempotency table in the CLI design doc reflects this posture:
  action verbs document their own idempotency (idempotent,
  not-idempotent, idempotent-on-same-input); `set` is uniformly
  idempotent. The distinction is load-bearing for scripting and cron
  use.
- Composes with
  [ADR002](ADR002-on-demand-cadence-no-daemon.md): on-demand
  invocations run action verbs as one-shot CLI commands; there is no
  daemon watching state and firing transitions on its own.
