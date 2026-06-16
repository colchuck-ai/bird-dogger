# ADR001 - Source/Hunt Decoupling

Two-axis decoupling at the Connection band: a Source authenticates a
connection to one external instance; a Hunt scopes a coverage set via
one or more `(source, selector)` entries. The same source can back
many hunts; one hunt can mix sources.

## Context

The Connection band has to answer two distinct questions:

1. **Who do we have credentials to talk to?** — one Jira Cloud
   instance, one Jira Data Center instance, future trackers, doc
   systems, message platforms. Each connection has its own auth, base
   URL, and availability state.
2. **What does a given coverage set include?** — projects, JQL
   queries, future source-type-specific selectors. The bird-dogger
   often watches one Jira instance under several different selector
   slices (per team, per release, per cross-team initiative), and may
   also mix slices across instances for a single cross-team chase.

The product README's `J001-O001-R007` and `J001-O002-R007` (Source
Registration) treat source registration as its own capability,
separate from the hunt-scope work. Conversely, `J001-O002-R001`
(Coverage Set Visibility), `J001-O002-R003` (Coverage Gap Discovery),
`J001-O002-R005` (Item Migration Linking), and `J001-O002-R006`
(Active Set Freshness) describe set-level coverage operations that
take the `(source, selector)` set as input but do not care how the
connection was authenticated.

Without a recorded decision here, the architecture could collapse
both axes into a single "watch" abstraction and force every coverage
selector to re-declare its auth, or force every auth to belong to
exactly one scope.

## Options

- **A. Decoupled Source and Hunt.** `C004` Source Registry holds
  per-instance auth and adapter binding; `C006` Hunt Registry holds
  named collections of `(source, selector)` entries. `C005` Source
  Adapter is instantiated against a registered source; `C008` Refresh
  Engine iterates a hunt's entries and asks the adapter per selector.
  Cost: two declaration verbs (`source`, `hunt`) instead of one
  combined `watch`; the bird-dogger learns the distinction up front;
  an item that appears in two hunts via the same source is one item
  under `C007`'s identity but two membership facts under `C016`.
- **B. Combined `watch` abstraction.** One declaration per "thing
  being watched," with auth and scope fused. Cost: a bird-dogger
  watching one Jira instance under three selector slices declares the
  auth three times (or the architecture introduces an implicit dedup
  that re-introduces the source concept under a different name);
  cross-source hunts become impossible without a separate composition
  primitive; PRD003's shared source registration loses its natural
  component home.
- **C. Hunt-owned source.** Each hunt owns its source declaration
  inline. Cost: same re-declaration cost as B, plus PRD003's
  two-outcomes-share-one-capability shape (one Jira registration
  usable from both an O001 slip-detection hunt and an O002
  coverage-gap hunt) becomes per-outcome configuration drift.

## Decision

Option A — decoupled Source and Hunt.

The two questions are genuinely distinct: auth lives with the
connection, scope lives with the hunt. The bird-dogger normally has
more hunts than sources, and PRD003's shared-source-registration
posture is realized cleanly only when source registration is its own
thing. The combined and hunt-owned alternatives both push cost onto
every multi-hunt bird-dogger and force the architecture to re-invent
a separate source concept under a different name.

The decoupling is also the structural backbone the rest of the
engineering layer assumes: `C005` Source Adapter is
source-type-agnostic and bound to a source, not a hunt; `C006`
validates source references against `C004`; `C008` receives a hunt
scope and asks `C005` per source.

## Consequences

- `C004` Source Registry and `C006` Hunt Registry stay separate
  components, each with their own CLI surface (`source` verbs and
  `hunt` verbs in `C001`). Two CRUD verb families to design and
  document.
- A multi-source hunt is a first-class shape: one hunt can mix Jira
  projects with future tracker selectors without architectural
  change. This anticipates the cross-team chase loop where a single
  initiative spans tools.
- PRD003's "one bundle, two outcome-layer requirements" lands
  cleanly: `C002` declaration + `C003` secret + `C004` registry +
  `C005` adapter contract is one implementation that satisfies both
  `J001-O001-R007` and `J001-O002-R007`.
- Source availability (`J001-O001-R006`) is reported per source by
  `C005`, not per hunt-membership. `C008` carries the per-hunt
  source-availability snapshot into `C016` so it can be rendered
  alongside the affected items in any hunt that uses the affected
  source.
- Item identity (`C007`) is source-scoped, not hunt-scoped: the same
  Jira issue surfaced in two hunts is one item under `C007` and two
  membership rows under `C016`. Item Migration Linking
  (`J001-O002-R005`) therefore applies once regardless of how many
  hunts the item participates in.
- The bird-dogger has to learn the source/hunt distinction during
  onboarding. CLI help text and any setup-wizard work must make the
  two-verb flow obvious; the alternative is a `watch` mental model
  the architecture rejects.
- Future source types (other trackers, doc systems, message
  platforms) plug in by implementing the `C005` Source Adapter
  contract; `C006` stays source-type-agnostic and selectors are
  source-type-specific shapes the adapter knows how to interpret.
  The Source Adapter contract shape itself is component-level ADR
  territory under `C005`.
