# ADR003 - Read-Only Against Sources

Source Adapters only read from external sources. Status overrides,
triage decisions, escalation timing recommendations, escalation
attempts, and routine touches are recorded locally. The actual
escalation message is sent by the bird-dogger out-of-tool.

## Context

Several requirements involve the bird-dogger producing per-item
judgments or recording actions, and a naive read would suggest the
product should propagate those back to the source of record:

- `J001-O003-R004` Status Override — the bird-dogger corrects an
  item's tracked status. Should the correction be written back to
  Jira?
- `J001-O005-R005` Escalation Record — the product records each
  escalation attempt with target, channel, time, and response. Should
  the product send the escalation (email, Slack, ticket comment)
  directly?
- `J001-O004-R004` Triage State Carry-Forward — prior triage
  decisions are recorded. Should they appear on the source's item as
  a comment or label?

Each R-doc describes the behavior in terms of local recording and
local surfacing; none asks for write-back. PRD004 (override
surfacing) explicitly frames the override as a local first-class
state that survives re-synthesis, with disagreement against the
source-derived reading surfaced for the bird-dogger to adopt or
dismiss — not as a write to the source that overwrites it.

Without a recorded decision, the engineering layer could be pulled
into write-back work: keeping Jira's "Status" field synchronized
with the Override Store, posting comments on escalation, writing
labels on triage decisions. That work is large, source-type-specific,
and introduces a new failure mode (the source rejects the write, or
accepts it then mutates it) that the local-only model avoids
entirely.

## Options

- **A. Read-only against sources.** `C005` Source Adapter exposes a
  read contract only: pull items, report availability. All overrides,
  triage decisions, touches, escalations, and recommendations are
  local writes to the State band; the bird-dogger sends actual
  messages (email, Slack, source comments) themselves, out-of-tool.
  `C013` Touch Log records what the bird-dogger did, not what the
  product sent. Cost: the source's status field can diverge from the
  bird-dogger's overridden status indefinitely; the source's history
  does not know an escalation went out; another consumer of the same
  source (a teammate, a dashboard, a report) sees the source's
  reading, not the bird-dogger's.
- **B. Write-back for status.** The Override Store mirrors status
  overrides back to the source. Cost: `C005` now needs a write
  capability, which means write-scope tokens (broader keyring entry,
  broader incident surface if a token leaks), source-type-specific
  write-conflict semantics (Jira workflows restrict which statuses
  can be set from which others — the override may be rejected), and
  a sync-loop question (when does re-assessment trust the source-side
  status that the product just wrote vs. treat it as the source's
  independent reading?).
- **C. Write-back for escalation.** The product sends the escalation
  message directly (Jira comment, SMTP email, Slack webhook). Cost:
  per-channel credential management, per-channel send-failure
  handling, per-channel template management, a notification subsystem
  the product otherwise avoids, plus the social cost that an
  escalation sent by a tool reads differently than one sent by the
  bird-dogger.
- **D. Full write-back.** Both B and C. Cost: union of the above.

## Decision

Option A — read-only against sources.

No requirement asks for write-back. PRD004's framing for overrides —
local first-class state with surfaced disagreement, not silent
replacement of source data — is a posture that write-back actively
breaks. The escalation flow is also explicitly framed as the
bird-dogger sending and the product recording (`J001-O005-R005`):
the product is a memory of what the bird-dogger did, not the actor.

Read-only also keeps the architecture cleanly bounded: one auth
scope per source (read-scope only), no per-channel notification
subsystem, no write-conflict semantics per source type. The Source
Adapter contract (`C005`) stays narrow, which makes adding future
source types tractable.

The cost — source-side data diverging from the bird-dogger's local
reading, another consumer of the same source not seeing the override
— is real and intentional. The product serves the bird-dogger's
chase loop on the bird-dogger's host. Sharing the corrected reading
with teammates is the bird-dogger's job, done out-of-tool through
whatever channel the team already uses.

## Consequences

- `C005` Source Adapter contract is read-only: pull items, report
  availability. The Jira implementation uses a read-scope API token;
  the OS keyring entry holds nothing that could write. This contains
  blast radius if a token leaks.
- `C010` Override Store and `C013` Touch Log are authoritative
  locally and have no upstream sync obligation. Re-assessment in
  `C009` treats the source's status as the source's reading, not as
  ground truth that supersedes the override; PRD004's "disagreement
  surfaced alongside" already specifies this composition.
- The bird-dogger sends escalation messages themselves through their
  normal channels (email, Slack, source comment, walk-up). `C013`
  records the attempt — target, channel, time, response — but the
  product does not perform the send. The CLI surface (`bdog touch
  add <item> --type escalation`) records the intent; see the
  [CLI design doc](../design-docs/cli.md) for the verb shape.
- No SMTP client, no Slack integration, no Jira write client, no
  per-channel template engine. The architecture's surface area stays
  at "read items, render decisions, record actions."
- Cross-bird-dogger visibility (another teammate seeing the same
  item's status in the same tool) is out of scope; the local-only
  constraint and read-only-against-sources together commit to
  single-user, single-host. A future requirement to share the
  bird-dogger's corrected view with teammates would supersede this
  decision with a new ADR that accepts the write-back or share-out
  cost.
- Composes with
  [ADR002](ADR002-on-demand-cadence-no-daemon.md): read-only access
  plus on-demand cadence keeps source rate-limit pressure low and
  predictable (one burst per checkpoint, no background polling).
  Source-side rate limits remain a real constraint at large hunt
  sizes; rate-limit handling is component-level ADR territory under
  `C005`.
