# ADR006 - Native Id Scheme

Native Birddog ids use `bdog<kind>-<n>` where `<kind>` is the entity
name and `<n>` is a monotonically increasing integer scoped per kind.
Items are `bdogitem-<n>`, notes `bdognote-<n>`, touches
`bdogtouch-<n>`. Source-derived items retain their source-provided id
as an alias; manual items have only the native id. Commands accepting
`<item>` resolve native ids unambiguously; source ids work when they
map to exactly one item across the State band — otherwise the command
fails fast with both matches listed and instructs the bird-dogger to
use the native id.

## Context

Every entity the CLI manages needs a stable, persistent identifier.
The bird-dogger edits the TOML config by hand in addition to using CLI
verbs; cross-entity references inside the TOML must key on something
that doesn't change when the bird-dogger renames a source, selector,
or hunt. Items, notes, and touches accumulate as SQLite state; the
State band (C007, C010, C012, C013, C016) must reference items without
ambiguity across sources.

Two identity pressures meet here. Declarative entities (sources,
selectors, hunts, contacts, chains) need stable ids so `--rename` is a
local field update with no propagation cost. Produced entities (items,
notes, touches) need ids that stay unique as items migrate across
source records and as multiple sources produce items whose source-side
keys may collide.

The CLI also accepts source ids as a convenience (`SUPPLY-1234` in
place of `bdogitem-42`), which forces the system to handle the
ambiguity case when two sources produce items with the same source-side
key. The resolution rule — fail fast, list matches, point at the native
id — must be uniform or the bird-dogger faces different behavior in
different code paths.

Without a recorded decision, different components could adopt different
id shapes, producing a mixed identity landscape the CLI cannot present
coherently and the bird-dogger cannot diagnose.

## Options

- **A. Kind-prefixed native scheme (`bdog<kind>-<n>`).** One shared
  prefix (`bdog`) names the binary; the kind segment disambiguates
  entity type at a glance in logs, error messages, and shell history.
  Integers are scoped per kind and monotonically increasing; the CLI
  is the canonical assigner. Source-derived items carry their
  source-provided id as an alias; ambiguous source ids fail fast.
  Cost: a per-kind counter; sequence gaps from deletions or hand-edits
  are possible and must be tolerated (ids are not reused).

- **B. UUIDs.** Globally unique, no counter state. Cost: unreadable in
  logs, error messages, and shell history; hand-editing TOML
  cross-references requires copy-pasting 36-character opaque strings;
  there is no way to distinguish an item id from a note id without a
  lookup; the bird-dogger has no freshness signal from the id itself.

- **C. Source-id-only, with collision handling.** Items use their
  source-provided id as the primary id; ambiguities fail fast. Cost:
  manual items have no source id and require a separate identity path,
  reintroducing the problem; source-side id changes or source
  deregistration leave dangling references in the State band; item
  identity is coupled to source health rather than to the local store.

- **D. Per-kind random tokens.** Short random strings scoped per kind
  (e.g., `item-x7k2`, `note-qr9f`). Cost: no human-readable freshness
  or order signal; collision avoidance requires re-roll logic; tokens
  in TOML cross-references are as opaque as UUIDs at TOML-editing
  time; the "which entity was created first" question requires a
  separate timestamp index.

## Decision

Option A — kind-prefixed native scheme.

The integer gives the bird-dogger a freshness signal at a glance:
`bdogitem-1` was captured before `bdogitem-42`. The kind prefix names
the entity type without a lookup: `bdognote-7` is a note, `bdogitem-7`
is an item. The shared `bdog` prefix scopes all ids to the binary's
namespace, making it immediately clear in error messages and shell
history which entity a reference belongs to.

Source ids work as convenient ad-hoc aliases for CLI invocations, but
the native id is the durable key the State band stores and
cross-references. Ambiguous source ids fail fast rather than silently
picking one — the correct behavior under
[PRD001](../../product/pdrs/PRD001-coverage-over-precision.md)'s
coverage-over-precision posture.

UUIDs (B) and random tokens (D) are both unreadable in practice; the
TOML-editing path and log-scanning path both require recognizable ids.
Source-id-only (C) creates a separate identity track for manual items
and ties item identity to source health rather than to the local store.

## Consequences

- All entities managed by the CLI carry a `bdog<kind>-<n>` id.
  Declarative entities: `bdogsource-<n>`, `bdogselector-<n>`,
  `bdoghunt-<n>`, `bdogcontact-<n>`, `bdogchain-<n>`. Produced
  entities: `bdogitem-<n>`, `bdognote-<n>`, `bdogtouch-<n>`. Each
  kind has its own monotonic counter scoped to the local SQLite/TOML
  store; integers are not globally unique across kinds.
- Source-derived items carry their source-provided id (e.g.,
  `SUPPLY-1234`) as an alias alongside `bdogitem-<n>`. Manual items
  have only the native id. Commands that take `<item>` accept either
  form; a source id that maps to more than one item fails fast with
  both matches listed so the bird-dogger uses the native id. Native
  ids are recommended in scripts; source ids stay convenient for
  ad-hoc invocations.
- Cross-entity references in the TOML config file key on the id, not
  the name. A hunt's selector list, a chain's contact members, and an
  item's chain or owner reference all carry ids. `bdog <noun> info`
  and `bdog <noun> list` print ids so the bird-dogger can wire
  cross-references by hand when editing the TOML directly.
- Sequence gaps from deleted entities are tolerated: the counter never
  resets; a deleted entity's id is never reused.
- Integers are not meaningful across kinds: `bdogitem-7` and
  `bdognote-7` are unrelated entities; the kind prefix is the only
  type disambiguator.
- `C007` (Item Store) is the identity hub for the State band. Every
  other State-band store — C010, C012, C013, C014, C016 — keys into
  items by `C007`'s `bdogitem-<n>`. An item that appears in two hunts
  via the same source is one `bdogitem-<n>` in C007 and two membership
  rows in C016. See
  [engineering README](../README.md) and
  [ADR001](ADR001-source-hunt-decoupling.md).
- `C002` (Config Store) owns the per-kind integer counters for
  declarative entities and is responsible for the assignment and
  write-back behavior on hand-added TOML entities. See
  [engineering README](../README.md).

### Hand-edit boundary

The TOML file is mutable both through the CLI and through a text
editor. The id field is the boundary between the two paths:

- **Adding an entity by hand:** write the entity's name, type, and
  value fields and omit `id`. The next CLI invocation that reads the
  TOML detects the missing id, assigns `bdog<kind>-<n>`, writes it
  back, and emits a one-line acknowledgement (`assigned
  bdogselector-7 to selector "supply-incidents"`). Wire up
  cross-references afterward — either via a CLI verb that resolves the
  name and writes the id, or by reading the printed id and editing the
  cross-reference list directly.
- **Editing an existing entity by hand:** renames and field changes are
  supported and equivalent to their CLI counterparts. Never hand-edit
  or delete the `id` field; the CLI is the canonical id assigner.
  A hand-written id that collides with an existing id or skips the
  sequence causes the next CLI invocation to fail fast and instruct
  the bird-dogger to remove the hand-written id and re-run so the CLI
  can assign a fresh one.
- **Cross-references use the id form:** name-or-id resolution is a
  CLI-argument convenience only; on-disk TOML cross-references must
  use the native id. `bdog <noun> info` and `bdog <noun> list` surface
  ids for this purpose. Full rules live in [CLI (C001)](../components/C001-cli.md#behavior) "Hand-edit reconciliation."
