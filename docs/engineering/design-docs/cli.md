# CLI Design

Specifies the `bdog` command surface: nouns, sub-verbs, flags,
sub-resource patterns, item identity, removal semantics, common
invocation sequences, and renderer surfaces. Realizes C001 (CLI)
and C015 (Renderer). Supersedes the placeholder verb list in the
[engineering root](../README.md).

The product is Birddog. The binary is `bdog`. Everything below
refers to the binary.

## Pattern

Every managed entity follows the same noun-first sub-verb shape:

```
bdog <noun> <sub-verb> [<args>] [<flags>]
```

The standard sub-verb set is `add | remove | info | list | set`.
Every top-level noun exposes all five. Two carve-outs apply:

- **Sub-resources with no state of their own.** `hunt selector`
  and `item depends-on` record only an association; the
  associated entity's data lives on its own noun. These expose
  `add | remove | list` only.
- **Action verbs.** Some nouns carry additional verbs that don't
  fit CRUD field-update semantics: `source test`, `source
  rotate-token`, `selector test`, `hunt bugel`, `hunt activate`,
  `hunt deactivate`, `item accept`, `item reject`, `item
  carry-forward`, `item confirm-owner`, `touch record-response`.
  These are documented under their noun.

`add` and `set` are strictly distinct: `add` errors on existing,
`set` errors on missing. No silent upsert. This applies uniformly,
including to composite-key resources like `chain member` and
`override` where the identity is `(parent, child)` or `(item,
field)`.

**Positional arguments on `add` differ by noun kind.** Declarative
nouns (`source`, `selector`, `hunt`, `contact`, `chain`) take the
bird-dogger-chosen name as the positional: `bdog source add
<name>`. Chase-state nouns (`item`, `touch`, `note`) get
auto-assigned `bdog<kind>-<n>` ids on `add`; their positional, if
any, is parent context (`bdog touch add <item>`, `bdog note add
<item>`), not a chosen name. The id is printed for subsequent
use.

Composite-key resources use positional arguments for the identity
pair: `bdog chain member add <chain> <contact>`, `bdog override
add <item> <field> <value>`. Multiplicity uses positional varargs
where the verb supports it: `bdog hunt selector add <hunt>
<selector>...`, `bdog hunt bugel [<hunt>...]`. Filter flags on
`list` accept a single value (`--hunt <hunt>`); to broaden the
filter, run the command per value or use `--all` where defined.

Clearing a field has two shapes, chosen by field type:

- **Typed reference fields** clear with the sentinel `none`:
  `bdog item set <item> --chain none`, `bdog item set <item>
  --owner none`. Reference targets are entities and `none` is
  reserved as a name, so the sentinel cannot collide with a real
  value.
- **Free-text fields** clear with `--clear-<field>`:
  `bdog contact set alice --clear-email`, `bdog contact set
  alice --clear-slack`. Channel handles and other free-text
  values could legitimately equal `"none"`, so the sentinel
  shape is unsafe there.

Sources and hunts are separate top-level nouns per
[ADR001](../adrs/ADR001-source-hunt-decoupling.md) (two-axis
decoupling); selectors are first-class per
[ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md).
Source registration is one shared capability per
[PRD003](../../product/pdrs/PRD003-shared-source-registration.md).

## Item identity

Native Birddog ids follow a single scheme: `bdog<kind>-<n>`, where
`<kind>` is the entity name and `<n>` is a monotonically increasing
integer scoped per kind. Items are `bdogitem-<n>`, notes are
`bdognote-<n>`, touches are `bdogtouch-<n>`. The shared `bdog`
prefix names the binary; the kind segment disambiguates entity
type at a glance in logs, error messages, and shell history.

Every item is assigned a `bdogitem-<n>` on first capture (manual)
or first refresh (source-derived). Source-derived items
additionally retain their source-provided id (e.g.,
`SUPPLY-1234`) as an alias. Manual items have only the native id.

Commands that take `<item>` accept either form:

- **Native id (`bdogitem-123`)** is unambiguous and always works.
- **Source id (`SUPPLY-1234`)** works when it resolves to exactly
  one item across the State band. If two items share the same
  source id (e.g., the same key from two different Jira instances,
  or from a Jira instance and a `local-json` file), the command
  fails fast with the matching items listed and instructs the
  bird-dogger to use the native id for the affected items.

This prevents silent cross-source collisions. Native ids are
recommended in scripts; source ids stay convenient for ad-hoc
invocations during a checkpoint.

Overrides and chain members are identified by composite keys and
have no standalone id.

## Renames

The bird-dogger types names; storage keys on ids. Every declared
entity (`source`, `selector`, `hunt`, `contact`, `chain`) carries
a stable internal id assigned on `add`, in addition to its
bird-dogger-chosen name. Cross-entity references in the TOML
configuration and the SQLite stores key on the id, not on the
name. The CLI resolves `<name>` to its id at command time.

Consequence: `--rename` is local. `bdog selector set supply-p0
--rename supply-priority` rewrites the selector's name while
every referencing hunt, item, and override stays pointed at the
same id. There is no propagation step and no risk of a
half-updated reference. Reusing a freshly-vacated name for a new
entity is safe — old references continue to point at the renamed
predecessor's id, not at the new entity.

`bdog <noun> info` and `bdog <noun> list` surface the id
alongside the name so a bird-dogger editing the TOML by hand can
match either form.

### Hand-edit reconciliation

The TOML config file is mutable from the CLI and from a text
editor. The boundary between the two paths is the `id` field.

**Adding a new entity by hand.** The bird-dogger writes the
entity's declaration with its name, type, and value fields and
*omits* the `id`. The next CLI invocation that reads the TOML
detects the missing `id`, assigns a `bdog<kind>-<n>`, writes it
back, and emits a one-line acknowledgement
(`assigned bdogselector-7 to selector "supply-incidents"`). Any
cross-reference to the new entity is wired up afterward — either
by the bird-dogger running a CLI verb that resolves the name and
writes the id (e.g., `bdog hunt selector add supply-q3
supply-incidents`), or by the bird-dogger reading the printed id
and hand-editing the cross-reference's id list directly.

**Editing an existing entity by hand.** Renames, field changes,
and removals via text editor are supported and equivalent to
their CLI counterparts. The `id` field must not be edited or
deleted by hand; the CLI is the canonical id assigner.
Hand-written ids that collide with existing ids or skip the
sequence cause the next CLI invocation to fail fast and instruct
the bird-dogger to remove the hand-written id and re-run so the
CLI can assign a fresh one.

**Cross-references must use the id form.** Per the
[engineering README](../README.md)'s C002 description,
cross-entity references inside the TOML key on the id, not the
name. A bird-dogger wiring up a new cross-reference by hand must
use the id (visible in `bdog <noun> info` and `bdog <noun> list`
output). The CLI's name-or-id input acceptance applies to
command-line arguments, not to the on-disk TOML cross-reference
fields.

## Removal semantics

Two postures:

- **Hard delete.** `touch remove` and `note remove` permanently
  drop the record. Used to correct a recording mistake; no
  history obligation.
- **Deactivate, preserve history.** `override remove` marks the
  override inactive but keeps it in history per
  [PRD004](../../product/pdrs/PRD004-override-surfacing.md);
  `override list --all` surfaces deactivated history.

For declarative entities and parents-of-references — `source`,
`selector`, `contact`, `chain`, `hunt`, and `item` (for manual
items) — removal is **no-cascade**: the bird-dogger must remove
references from consumers first, then remove the dependency.
Cascading deletes would let one verb silently destroy chains of
declarations and per-item history; failing fast keeps the
bird-dogger in control of the teardown.

Concretely:

| Removing | Required pre-step |
|---|---|
| `source` | Remove or re-source every selector that references it. |
| `selector` | Remove the selector from every hunt that references it (via `hunt selector remove`). |
| `contact` | Remove the contact from every chain that references it (via `chain member remove`), and clear any item owner reference (via `item set --owner none`). |
| `chain` | Clear the chain reference from every item that references it (via `item set --chain none`). |
| `hunt` | Reassign or remove every manual item whose `--hunt` is this hunt (via `item set --hunt <other>` or `item remove`). Source-derived items disappear from coverage automatically. |
| `item` (manual) | None; manual items have no downstream references. (Source-derived items cannot be removed individually — they are managed via the owning selector.) |

`remove` invocations check for outstanding references and fail
with a list of the blocking references and the verbs that would
clear them.

## Idempotence

| Verb shape | Re-invocation behavior |
|---|---|
| `add` | Not idempotent. Second `add` for the same identity errors. |
| `set` | Idempotent on identical input. Setting a field to its current value succeeds and is a no-op. |
| `remove` (declarative entities, manual items, touches, notes) | Not idempotent. Removing a non-existent or already-removed entity errors with `not found`. |
| `override remove` | Idempotent. Removing an already-deactivated override succeeds as a no-op; history is preserved per PRD004 either way. |
| `info`, `list`, `*-test` | Pure reads. Idempotent by construction. |
| `hunt bugel` | Idempotent in semantics, not in side effects. Each invocation refreshes, re-assesses, synthesizes, and writes the current snapshot. Two back-to-back bugels against the same sources produce the same rendered output if no source-side data changed; assessment timestamps still advance. |
| `hunt bugel --no-refresh` | Idempotent on identical State-band input. Re-synthesizes against current State without pulling sources. |
| `item accept` / `item reject` / `item carry-forward` | Not idempotent across state transitions. Re-issuing the same coverage verb on an item already in the target state errors with `already <state>`. |
| `touch record-response` | Idempotent on identical input; corrective on different input. Re-invocation always sets the touch's response to the supplied values, overwriting any prior response. |
| `hunt activate` / `hunt deactivate` | Idempotent. Activating an already-active hunt (or deactivating an already-inactive one) succeeds as a no-op. |
| `item confirm-owner` | Each invocation advances the owner-confirmation timestamp; the side effect is intentional. Not idempotent. |

The general posture: queries and identity-shape edits are
idempotent; state-machine transitions and timestamp-advancing
actions are not. Idempotent verbs are safe in scripts and
schedulers (cron / launchd / systemd timers per
[ADR002](../adrs/ADR002-on-demand-cadence-no-daemon.md)).

## Global flags

Apply to every command:

```
--output, -o  text|json    default: text
--quiet,  -q               suppress headers and labels; data only
```

## Timestamps

All timestamps the CLI accepts (`--at`, `--responded-at`,
`--due`, and similar) are parsed as ISO 8601 in the host's local
zone unless the value carries an explicit offset (`Z`, `+02:00`).
Date-only fields like `--due` accept `YYYY-MM-DD`; datetime
fields accept `YYYY-MM-DDTHH:MM[:SS][offset]`.

Relative renderings produced by the renderer (`2h ago`, `3d
ago`, `not-yet`) derive from the host's wall clock at render
time. Absolute renderings emit ISO 8601 in the host's local zone
with the offset included.

## Nouns

### source

A source is a connection to one external system instance. It holds
the type, base URL or file path, and (when the type carries one) a
reference to its secret in the OS keyring. Secrets never appear as
CLI flags; `source add` (for token-bearing types) and `source
rotate-token` always prompt interactively. The
`BDOG_SOURCE_<NAME>_TOKEN` environment variable provides the
headless fallback for CI; `<NAME>` is the source name uppercased
with `-` replaced by `_` (`ttd-jira` → `BDOG_SOURCE_TTD_JIRA_TOKEN`).
Source registration is one shared capability per
[PRD003](../../product/pdrs/PRD003-shared-source-registration.md).

`local-json` is a first-class source type per
[ADR005](../adrs/ADR005-local-json-source-type.md). It is backed
by a local file (`--path`), requires no token, and reports
availability from the file's readability and parseability.

```
bdog source add <name>
    --type    jira-cloud|jira-dc|local-json    required
    --url     <base-url>                       required for remote types
    --path    <file-path>                      required for local-json

bdog source remove <name>
bdog source info   <name>
bdog source list

bdog source set <name>
    --rename  <new-name>
    --url     <base-url>
    --path    <file-path>

bdog source test <name>
    --verbose                       show response details, not just pass/fail

bdog source rotate-token <name>     interactive prompt; writes to keyring
```

`source set` rejects `--url` on a `local-json` source and `--path`
on a remote source; type is fixed at `add` time. To change the
type of an existing source, remove it (after clearing referencing
selectors) and re-add.

`source remove` is rejected if any selector references the source;
the error lists the blocking selectors.

### selector

A selector is a named, reusable query against one source. Per
[ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md),
selectors are first-class declarations; the same selector can be
referenced by multiple hunts, and editing a selector propagates
to every referencing hunt on the next bugel that touches them.

```
bdog selector add <name>
    --source         <source-name>           required
    --selector       <value>                 required
    --selector-type  project|jql|filter      required

bdog selector remove <name>
bdog selector info   <name>
bdog selector list
    --source         <source-name>           filter to one source
    --selector-type  project|jql|filter      filter to one type

bdog selector set <name>
    --rename         <new-name>
    --source         <source-name>
    --selector       <value>
    --selector-type  project|jql|filter

bdog selector test <name>
    --verbose                                show resolved query, sample matches, and source response detail
```

`--selector-type` is required on `selector add`; it is not
inferred from the value shape. Explicit typing keeps selector
definitions unambiguous and avoids surprises when a value happens
to parse as more than one type.

`selector remove` is rejected if any hunt references the selector;
the error lists the blocking hunts and the `bdog hunt selector
remove` invocations that would clear them.

### contact

A contact is a person the bird-dogger interacts with — an item
owner, an escalation chain member, a touch target. Contacts are
declared once and referenced from chains and items by name.

```
bdog contact add <name>
    --email   <email>
    --slack   <handle>
    --jira    <username>
    --about   <text>     stable freetext description of the contact

bdog contact remove <name>
bdog contact info   <name>
bdog contact list

bdog contact set <name>
    --rename        <new-name>
    --email         <email>
    --slack         <handle>
    --jira          <username>
    --about         <text>
    --clear-email                  clear the email handle while preserving the contact
    --clear-slack                  clear the slack handle
    --clear-jira                   clear the jira handle
    --clear-about                  clear the contact description
```

`--about` is a stable freetext property of the contact — distinct
from the first-class `bdog note` records (those attach to items
only, per C012), from `bdog override --reason` (per-write
override justification), and from `bdog touch --message`
(communication content). Channel-handle flags and `--about` are
free text; clearing uses `--clear-<field>` per the §Pattern
sentinel rule. The shorter `<flag> none` sentinel applies only
to typed reference fields (`--chain`, `--owner`), where `none`
is reserved as an entity name and cannot collide with a real
value. `--clear-*` flags and their value-bearing siblings are
mutually exclusive in one invocation (e.g., `--email foo@bar
--clear-email` errors).

`contact remove` is rejected if any chain references the contact
or any item has the contact as its owner; the error lists the
blocking references and the verbs that would clear them.

### chain

A chain is a named, reusable escalation contact list. Items
reference a chain by name via `item set --chain`; editing the
chain propagates to all items using it. Members are ordered by
position (1-indexed, no gaps). Position 1 is the primary contact.
Escalation walks the list upward from lower to higher positions.

```
bdog chain add <name>
bdog chain remove <name>
bdog chain info   <name>
bdog chain list

bdog chain set <name>
    --rename  <new-name>
```

`chain remove` is rejected if any item references the chain; the
error lists the blocking items.

#### chain member

Members are contacts ordered by position. `add` inserts at the
given position and shifts higher-numbered members down; `remove`
closes the gap automatically; `set --position` moves a member and
shifts others to maintain a contiguous list.

```
bdog chain member add <chain> <contact>
    --position  <n>     insert at n, shift n+ down; omit to append

bdog chain member remove <chain> <contact>

bdog chain member info <chain> <contact>

bdog chain member list <chain>
    # renders in order: 1. alice  2. bob  3. carol

bdog chain member set <chain> <contact>
    --position  <n>     move to n; shifts others to close old gap
```

`chain member add` errors if the contact is already a member of
the chain; `chain member set` errors if the contact is not a
member. The contact must exist in the Contact Registry first; the
error from `chain member add` lists the `bdog contact add`
invocation that would resolve a missing contact.

### hunt

A hunt is a named collection of selector references. It is the
unit of bugel scoping. Hunt metadata is minimal — the query
content lives on the associated selectors. Hunts carry an active
flag; `bdog hunt bugel` with no arguments operates on active
hunts only.

```
bdog hunt add <name>
bdog hunt remove <name>
bdog hunt info   <name>           shows metadata + associated selectors + active flag
bdog hunt list   [--all | --inactive]
                                  default: active only; --inactive shows inactive only;
                                  --all shows both

bdog hunt set <name>
    --rename      <new-name>

bdog hunt activate   <name>       sets active = true; idempotent
bdog hunt deactivate <name>       sets active = false; idempotent

bdog hunt bugel [<hunt>...]
    --no-refresh                  skip the data pull; re-synthesize on current state
    --dry-run                     report what would be fetched and rendered without writing
    --all                         include inactive hunts (with no <hunt> args)
    --verbose                     show per-selector pull detail and per-source availability detail
```

`hunt bugel` with no arguments runs the bugel across all active
hunts. Explicit naming overrides the active filter — `bdog hunt
bugel old-q2` runs the bugel for `old-q2` even when it is
inactive. The bugel is the only refresh path; partial work is
covered by `--no-refresh` (skip the pull, re-synthesize current
State). On-demand cadence per
[ADR002](../adrs/ADR002-on-demand-cadence-no-daemon.md): no
background process, the bird-dogger drives the schedule.

`hunt remove` is rejected if any manual item is assigned to the
hunt via `item add --hunt` (or `item set --hunt`); the error
lists the blocking items.

#### hunt selector

Membership management for the selector ↔ hunt association. The
selector's source, query, and type live on the `selector` noun;
this sub-resource only records the association.

```
bdog hunt selector add    <hunt> <selector>...     errors if any already present
bdog hunt selector remove <hunt> <selector>...
bdog hunt selector list   <hunt>
```

`hunt selector add` accepts one or more selector names as
positional varargs; each association is recorded independently. If
any named selector is already associated with the hunt, the
command errors with the conflicting names; the partial state is
not written.

The hunt-selector association is a set, not an ordered list. The
refresh engine walks all referenced selectors per bugel; ordering
between them is not bird-dogger-visible and not bird-dogger-
controlled.

### item

An item is a tracked open work unit. Items are either
source-derived (pulled by a selector during refresh) or manually
captured. Manual items are first-class: they participate in slip
assessment, urgency ranking, bugel ordering, override, and
escalation.

`item add` is for manual items only — it carries no URL or source
reference. To pull a specific remote item not covered by an
existing selector, create a selector targeting it (e.g., JQL
`issue = PROJ-123`) and associate it with a hunt.

```
bdog item add
    --title       <text>      required
    --hunt        <hunt>      required
    --due         <date>
    --first-note  <text>      convenience: creates a bdognote-N record on the new item,
                              equivalent to a follow-up `bdog note add <new-item> --text <text>`

bdog item remove <item>
    # manual items only; source-derived items are rejected with a
    # pointer to the owning selector

bdog item info <item>
    --show-basis        expand assessment inference inputs
    --show-history      include override, touch, and note history

bdog item list
    --hunt    <hunt>
    --sort    urgency|due|freshness|slip-age    default: urgency

bdog item set <item>
    --title              <text>            manual items only; source-derived items are rejected
                                           (title changes belong at the source, not locally)
    --trajectory         due-date|pace|none
    --due                <date>
    --pace               <text>            freeform, for pace-style trajectories
    --hunt               <hunt>            manual items only; reassign to a different hunt
    --chain              <chain>|none      assign or clear escalation chain
    --owner              <contact>|none    assign or clear item owner
    --clear-pace                           clear the pace text without changing trajectory
```

Trajectory cross-validation: setting `--trajectory due-date`
requires `--due` to be set (now or previously); setting
`--trajectory pace` requires `--pace` to be set; setting
`--trajectory none` clears both `--due` and `--pace`. Setting
`--due` on a `pace` trajectory or `--pace` on a `due-date`
trajectory errors with the conflicting trajectory and the
`bdog item set --trajectory <other>` invocation that would
resolve it.

Coverage actions on items (operate on items in candidate or
drop-off state per hunt; see "Renderer surfaces" for how these
states surface in the bugel):

```
bdog item accept        <item> --hunt <hunt>    move a candidate into the active set
bdog item reject        <item> --hunt <hunt>    mark a candidate rejected for this hunt
bdog item carry-forward <item> --hunt <hunt>    keep a drop-off in the active set
bdog item confirm-owner <item>                  re-record owner-confirmation timestamp
```

`item confirm-owner` lives on its own verb rather than as a flag
on `set`: the bird-dogger is recording a fact about the world (I
just checked with the owner) rather than editing a field. Each
invocation advances the confirmation timestamp; the side effect
is intentional.

Inter-item dependencies are a sub-resource:

```
bdog item depends-on add    <item> <other-item>...
bdog item depends-on remove <item> <other-item>...
bdog item depends-on list   <item>
```

### override

An override is a bird-dogger correction to a produced judgment.
Covers three judgment fields: `status` (from C009 Assessment
Engine), `intervention` (from C011 Synthesis Engine), and
`escalation-timing` (from C011 Synthesis Engine). Overrides
persist across re-synthesis; new synthesis output is surfaced
alongside the active override for adoption or dismissal. `remove`
deactivates an override — history is preserved and remains
recoverable per
[PRD004](../../product/pdrs/PRD004-override-surfacing.md).

The identity of an override is the `(item, field)` pair; both are
positional.

```
bdog override add <item> <field> <value>
    --reason  <text>     why this override is being set; embedded on the override record

bdog override remove <item> <field>

bdog override info <item> [<field>]
    # without <field>: shows all fields for the item

bdog override list
    --item    <item>
    --field   status|intervention|escalation-timing
    --all                                   include deactivated history (default: active only)

bdog override set <item> <field> <value>
    --reason  <text>
```

`add` errors if an active override already exists for the
`(item, field)` pair; `set` errors if no active override exists.

`--reason` is the bird-dogger's justification for the override
(why the source-side reading is wrong, or why this judgment was
intervened). It is embedded on the override record; for
open-ended triage thinking that should live on its own, write a
`bdog note add` instead.

Field values:

| Field | Value vocabulary |
|---|---|
| `status` | Any status valid for the source type (closed-vocabulary per source); freeform for manual items |
| `intervention` | `intervene-now` \| `not-now` \| `cannot-recommend` |
| `escalation-timing` | `escalate-now` \| `not-yet` \| `cannot-recommend` |

`status` for manual items is freeform because there is no source
workflow to draw a vocabulary from. The renderer treats freeform
status values readably (see "Renderer surfaces"); the bird-dogger
can keep a consistent personal vocabulary by convention.

### touch

A touch is a recorded bird-dogger contact event. It captures
target, channel, time, type, message, and optional response.
Touches with type `escalation` are distinguished from `checkin`
touches in synthesis and display but share the same record shape.
The
response window (elapsed since last touch) is derived from the log
at read time, not stored. `touch remove` hard-deletes — it
corrects a recording mistake and carries no history obligation.
The product records what the bird-dogger did; the bird-dogger
sends the actual message out-of-tool per
[ADR003](../adrs/ADR003-read-only-against-sources.md).

```
bdog touch add <item>
    --type      escalation|checkin            required
    --channel   slack|email|jira-comment|in-person|...    required
    --target    <contact>
    --message   <text>                        what was communicated; embedded on the touch record
    --at        <datetime>                    default: now

bdog touch remove <id>

bdog touch info <id>

bdog touch list
    --item      <item>
    --since     <datetime>
    --channel   <channel>
    --type      escalation|checkin
    --open                                    touches with no recorded response

bdog touch set <id>
    --channel   <channel>
    --target    <contact>
    --at        <datetime>
    --type      escalation|checkin
    --message   <text>

bdog touch record-response <id>
    --response       <text>                   required
    --responded-at   <datetime>               required
```

`--type` is required on `touch add`. Forcing the bird-dogger to
name the touch's intent at creation keeps the
escalation/routine distinction reliable downstream — the synthesis
engine and renderer treat `escalation` as a first-class signal.

`touch set` is corrective for the touch itself: a touch recorded
with the wrong channel, target, time, type, or message can be
fixed in place rather than removed and re-added.

`touch record-response` is its own verb because recording a
response is a distinct event with its own timestamp, not a field
update on the originating touch. Both `--response` and
`--responded-at` are required; the bird-dogger names when the
response actually arrived rather than defaulting to wall-clock
now and silently mis-stamping a back-filled response. Re-invoking
`record-response` overwrites the prior values — the verb is the
corrective path for an entered-wrong response as well as the
first-time recording path; a no-op repeat of identical input is
safe.

`--target` resolves to a registered contact when set. To record a
touch with an unregistered person, either add the contact first
(`bdog contact add`) or leave `--target` unset and capture the
person in `--message`.

`--message` is the touch's content (what was communicated). It
is distinct from a first-class `bdog note` (open-ended triage
thinking on the item, attached separately and surfaced
alongside) and from `bdog override --reason` (justification for
a judgment override). Capturing a longer post-touch reflection
that should outlive the touch itself belongs on a note, not in
`--message`.

### note

A note is a per-item open-vocabulary annotation: the bird-dogger's
working thought during triage. Distinct from `override` (closed-
vocabulary judgment correction) and from `touch` (contact event).
Realizes C012 Note Store; each note auto-captures a snapshot of
context at write time (rank, signals, recommendation, active
overrides) so later readers see both the note and what was visible
when it was written. `note remove` hard-deletes — same posture as
`touch`.

```
bdog note add <item>
    --text  <text>     required

bdog note remove <id>

bdog note info <id>

bdog note list
    --item    <item>
    --since   <datetime>

bdog note set <id>
    --text  <text>     required
```

Text is the only mutable field on a note today; the flag form
matches every other `add`/`set` shape and leaves room for future
fields (`--tag`, `--severity`) without re-shaping the verb.

Notes are append-then-correctable: most are written once during
a checkpoint and never edited.

## Common workflows

### First-time setup

```sh
bdog source add ttd-jira --type jira-cloud --url https://jira.example.com
bdog source test ttd-jira

bdog selector add supply-open \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND status != Done AND sprint in openSprints()"

bdog selector add supply-p0 \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND priority = P0 AND status != Done"

bdog selector test supply-open
bdog selector test supply-p0

bdog hunt add supply-q3
bdog hunt selector add supply-q3 supply-open supply-p0

bdog hunt bugel supply-q3
```

### Daily bugel

```sh
# full cycle across all active hunts
bdog hunt bugel

# scoped to one hunt
bdog hunt bugel supply-q3

# re-synthesize without a data pull
bdog hunt bugel supply-q3 --no-refresh

# scan the triage list scoped to one hunt
bdog item list --hunt supply-q3
```

### Recording a touch and its response

```sh
bdog touch add bdogitem-1234 \
    --type checkin \
    --channel slack \
    --target alice \
    --message "asked for updated ETA on auth integration"

# the touch's id (bdogtouch-N) is printed; record-response is its own verb
bdog touch record-response bdogtouch-87 \
    --response "landing Friday, blocked on code review from bob" \
    --responded-at "2026-06-17T14:30:00"
```

### Adding a note during triage

```sh
bdog note add bdogitem-1234 \
    --text "alice flagged this in standup; bob is the code-review blocker. \
            watching for friday landing."

# later: list this item's notes (newest first)
bdog note list --item bdogitem-1234
```

### Overriding a status judgment

```sh
bdog item info bdogitem-1234 --show-basis

bdog override add bdogitem-1234 status blocked \
    --reason "alice confirmed in standup; jira status is stale"

# clear once the source catches up
bdog override remove bdogitem-1234 status
```

### Setting up contacts, a chain, and escalating

```sh
bdog contact add alice --email alice@example.com --slack alice --jira alice
bdog contact add bob   --email bob@example.com   --slack bob   --jira bob
bdog contact add carol --email carol@example.com --slack carol --jira carol

bdog chain add supply-leadership
bdog chain member add supply-leadership alice    # position 1
bdog chain member add supply-leadership bob      # position 2
bdog chain member add supply-leadership carol    # position 3

bdog item set bdogitem-1234 --owner alice --chain supply-leadership

bdog touch add bdogitem-1234 \
    --type escalation \
    --channel email \
    --target bob \
    --message "bdogitem-1234 at risk for q3; alice unresponsive since Monday"
```

### Capturing a manual item

```sh
bdog item add \
    --title "partner API rate limit increase" \
    --hunt supply-q3 \
    --due 2026-07-15 \
    --first-note "committed in q3 planning; tracked by dave's team, not in jira"

# the item's id (bdogitem-N) is printed; use it on subsequent commands
bdog item set bdogitem-487 --trajectory due-date --due 2026-07-15 --owner dave
```

### Adding a selector to a live hunt

```sh
bdog selector add supply-incidents \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND issuetype = Incident AND status != Done"

bdog selector test supply-incidents
bdog hunt selector add supply-q3 supply-incidents

# preview what the next bugel will pull and render
bdog hunt bugel supply-q3 --dry-run

# run the bugel for real
bdog hunt bugel supply-q3
```

### Reusing a selector across two hunts

```sh
bdog hunt add infra-q3
bdog hunt selector add infra-q3 supply-p0

# broaden scope; propagates to both hunts on next bugel
bdog selector set supply-p0 \
    --selector "project in (SUPPLY, INFRA) AND priority = P0 AND status != Done"
```

### Accepting a coverage candidate

```sh
# the bugel surfaced bdogitem-981 as a candidate (in a watched
# source, not yet in the active set for this hunt)
bdog item accept bdogitem-981 --hunt supply-q3

# or, mark it as deliberately out of coverage for this hunt
bdog item reject bdogitem-981 --hunt supply-q3
```

### Carrying a drop-off forward

```sh
# the bugel surfaced bdogitem-714 as a drop-off (left the active
# set since last refresh, e.g., closed at source but bird-dogger
# knows the chase isn't done)
bdog item carry-forward bdogitem-714 --hunt supply-q3
```

### Deactivating a hunt at end of quarter

```sh
bdog hunt deactivate supply-q3

# inactive hunts are skipped by default; explicit invocation still works
bdog hunt bugel supply-q3   # runs supply-q3 even though inactive

# later, reactivate
bdog hunt activate supply-q3
```

### Rotating a source token

```sh
bdog source rotate-token ttd-jira
# interactive prompt for the new token; written to keyring
bdog source test ttd-jira
```

### Tearing down a source (no-cascade)

```sh
# fails: blocking selectors are listed
bdog source remove ttd-jira
# Error: source `ttd-jira` is referenced by 3 selectors:
#   - supply-open  (run: bdog selector remove supply-open)
#   - supply-p0    (run: bdog selector remove supply-p0)
#   - supply-incidents
# Remove the selectors first (each may itself require removing
# from hunts first).
```

## Renderer surfaces

The bugel and item-info output shapes are part of the design,
not implementation detail. C015's rendering obligations
(PRD001 readability, PRD002 two-timestamp distinctness, PRD004
override marker handling) and C011's list-level signal-selection
obligation (J001-O004-R002 — which urgency-driving facts appear
on each row) both have concrete commitments here.
This section names the shape of each rendering verb's output so
the obligations from
[PRD001 (coverage over precision)](../../product/pdrs/PRD001-coverage-over-precision.md),
[PRD002 (two-timestamp freshness)](../../product/pdrs/PRD002-per-item-freshness-presentation.md),
and
[PRD004 (override surfacing)](../../product/pdrs/PRD004-override-surfacing.md)
are concrete in the design, not implicit.

### `bdog hunt bugel`

The primary triage surface. Composed sections, in order:

1. **Header.** Hunt name(s), the set-level last-refresh timestamp
   (PRD002 cousin shape; per-hunt when multiple hunts run), and
   a one-line source-availability summary (`2/3 sources
   available`).
2. **Source availability detail** (only when any source is
   unavailable). Lists the unavailable sources, the reason
   (network-error, file-missing, unparseable, auth-failed), and
   the affected hunts. Items from unavailable sources surface in
   the main table marked `source-unavailable` per
   `J001-O001-R006` — never blank, never assumed-healthy.
3. **Active items table**, ordered by urgency rank. Columns:
   - `id` — native id (`bdogitem-N`); source id shown as `(SUPPLY-1234)`
     suffix when present.
   - `title` — truncated.
   - `slip` — slip status. Accepts uncertainty states
     `cannot-assess` and `not-yet-assessed` as first-class values
     per PRD001.
   - `status` — status reading. When an override is active, the
     override value renders as the displayed status with a `*`
     marker; the inferred reading renders in parentheses when it
     differs (PRD004 disagreement surface). Uncertainty states
     (`cannot-determine`, `sources-disagree`) are first-class
     values per PRD001.
   - `intervention` — recommendation. Same override-marker and
     uncertainty rules apply (`cannot-recommend`).
   - `data` — last-underlying-data-change timestamp, relative
     (`2h ago`, `3d ago`); literal `unknown` when the source does
     not expose one (PRD001 + PRD002).
   - `assessed` — last-assessed timestamp, relative; literal
     `not-yet` when never assessed.
   - `owner` — current owner's display name; `!` suffix when the
     source-derived owner differs from the registered owner
     (PRD001 disagreement on contacts); `?` suffix when
     owner-confirmation is older than a configurable freshness
     window (J001-O005-R002 Owner Currency).
4. **Notes recap** (collapsed by default; expanded with
   `--show-notes` once the flag is added — out of scope for now).
   At minimum, the table marks rows with a `note` indicator when
   the item has any active notes; the bird-dogger drills into
   `bdog item info` to read them.
5. **Drop-offs.** Items that left the active set since the
   previous refresh. Each row shows `id`, `title`, last status,
   reason (`closed-at-source`, `selector-no-longer-matches`,
   `source-removed`). PRD001 obligation: drop-offs are a
   distinct section, never silently disappeared.
6. **Item candidates.** Items in watched sources excluded from
   the active set, with `id`, `title`, source, and the selector
   that surfaced them as adjacent. The bird-dogger accepts or
   rejects via `bdog item accept` / `bdog item reject`.
7. **Source candidates.** Sources referenced from watched data
   but not registered (e.g., a Jira issue mentioning another
   project's key). Listed with the reference text and the hunt
   that observed it. The bird-dogger acts via `bdog source add`
   or ignores; no inline dismiss verb in v1.

### `bdog item info`

Stacked sections:

- **Identity.** Native id, source id (if any), origin (manual or
  source-derived), title, owning hunt (for manual items), current
  active overrides (one line per field).
- **Freshness.** Two timestamps labeled distinctly: `last data
  change` and `last assessed`. Both shown with absolute and
  relative formats. PRD002 obligation: these are never merged
  into a single "last updated."
- **Slip & status.** Current slip reading, status reading, basis
  summary. With `--show-basis`, expands to per-source inputs and
  any disagreement detail; uncertainty states render with their
  explanation ("not yet assessed because trajectory not
  recorded").
- **Overrides.** For each field with any history: current state
  (active or inactive), value, when set, reason, and the current
  inferred reading from synthesis when it differs (PRD004
  surface). With `--show-history`, expands to full override
  history including deactivated entries.
- **Recommendations.** Intervention and escalation-timing
  recommendations, with `cannot-recommend` as a first-class
  value and active overrides surfaced alongside.
- **Owner & chain.** Current owner, last-confirmed-at;
  source-derived owner when it differs (PRD001 disagreement
  surface). Chain name and ordered members.
- **Dependencies.** Inter-item dependencies (`depends-on`) and
  reverse dependencies (`depended-on-by`).
- **Recent notes.** Last N notes (default: 5) newest-first, each
  with text, write timestamp, and one-line context snapshot.
  `--show-history` includes all notes.
- **Recent touches.** Last N touches (default: 5) newest-first,
  each with type, channel, target, time, message, response,
  derived response window. `--show-history` includes all touches.

### `bdog item list`

Same column set as the bugel's active-items table; no header or
drop-off/candidate sections. Intended for ad-hoc scanning when
the bird-dogger doesn't need the full bugel composition.

### `bdog hunt info` / `bdog hunt list`

- `bdog hunt info <hunt>` shows hunt name, active flag,
  set-level last-refresh timestamp, the associated selectors
  (with each selector's source and type for context),
  per-source availability state from the most recent refresh,
  and a count summary (active items, drop-offs since last
  refresh, candidate items, candidate sources).
- `bdog hunt list` shows one row per hunt: name, active flag,
  last-refresh, active-item count.

### Other list/info verbs

`source`, `selector`, `contact`, `chain`, `override`, `touch`,
`note` `list` and `info` follow conventional CRUD rendering —
labeled fields for `info`, columnar table for `list` with
identity column first. Uncertainty rendering does not apply to
declarative entities (`source`, `selector`, `contact`, `chain`)
because they carry no produced readings; it does apply to
`override` and `touch` and `note` only insofar as they reference
items whose readings could be uncertain.

## Design decisions

**Noun-first sub-verb pattern.** Every managed entity follows
`bdog <noun> <sub-verb>`. Top-level nouns expose the full
`add | remove | info | list | set` set; stateless sub-resources
(`hunt selector`, `item depends-on`) expose `add | remove | list`
only. Learning one noun transfers to all others; tab completion
surfaces a consistent shape across the entire binary.

**`add` errors on existing, `set` errors on missing.** No silent
upsert. Composite-key resources (`chain member`, `override`,
`hunt selector`) inherit the same rule against their composite
identity. The bird-dogger always knows which posture they're
asking for.

**Storage keys on stable ids; the CLI speaks names.** Every
declared entity carries a `bdog<kind>-<n>` id alongside its
bird-dogger-chosen name. Cross-entity references in the TOML
config and SQLite stores key on the id, not the name. Renames
are therefore local — there is no propagation step and no
half-updated reference. The CLI accepts the name on input and
prints the id in `info` and `list` for hand-editing the TOML.

**Native `bdog<kind>-<n>` ids; source ids accepted with
collision-fail.** See
[ADR006](../adrs/ADR006-native-id-scheme.md) for the full
decision and rationale. In brief: items, notes, and touches each
carry their own native id under one scheme — `bdogitem-<n>`,
`bdognote-<n>`, `bdogtouch-<n>` — with source-derived items also
carrying their source id as an alias. Commands accept either form;
ambiguous source ids fail fast with matches listed.

**Selectors are top-level, not hunt-owned.** A selector's
definition (source, query, type) is independent of which hunts
use it. Per
[ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md),
lifting selectors to a first-class noun lets the same query
appear in multiple hunts and lets changes propagate without
per-hunt edits.

**Contacts are top-level; chains reference contacts.** A contact
is the person; a chain is a named ordered list of contacts; an
item references one contact for ownership and optionally one
chain for escalation. Lifting contacts out of chains lets the
same person appear in multiple chains and as multiple items'
owner without re-declaration; channel handles, freshness, and
disagreement state live with the contact.

**`hunt selector` exposes only `add | remove | list`.** The
sub-resource records only membership — there is nothing to `set`
or inspect via `info` that does not already live on the
`selector` noun. The asymmetry with `chain member` is
intentional: `chain member` carries its own state (position) and
its order is bird-dogger-visible; `hunt selector` is a set, not
an ordered list, and the refresh engine walks all referenced
selectors per bugel without bird-dogger-controlled ordering.

**Manual items carry no source reference.** `item add` accepts
no URL or source identifier. A manual item is a purely local
record. Tracking a specific remote item that falls outside
existing selectors is done by creating a targeted selector —
this keeps item identity clean and avoids a hybrid manual-plus-
remote category.

**No-cascade removal.** Removing a declarative entity (`source`,
`selector`, `contact`, `chain`, `hunt`) is rejected when any
consumer still references it. The error names the blocking
references and the verbs that would clear them. Cascade would
let one verb silently destroy chains of declarations and
per-item history; failing fast keeps the bird-dogger in control.
See [ADR007](../adrs/ADR007-no-cascade-removal.md) for the full
decision and rationale.

**Two clearing shapes: `none` for typed references, `--clear-<field>`
for free-text.** `bdog item set <item> --chain none` and
`--owner none` clear typed reference fields where `none` is
reserved as an entity name and cannot collide with a real value.
`bdog contact set <name> --clear-email` clears a free-text
channel handle where `"none"` could legitimately be a real
value. Choosing the shape by field type keeps the sentinel safe
where it is safe and unambiguous where it isn't. See
[ADR011](../adrs/ADR011-two-clearing-shapes.md) for the full
decision and rationale.

**`override remove` deactivates; `touch remove` and `note
remove` hard-delete.** Overrides are deactivated on remove —
history is preserved per
[PRD004](../../product/pdrs/PRD004-override-surfacing.md).
Touches and notes are hard-deleted on remove — they correct a
recording mistake and carry no equivalent history obligation.
See [ADR008](../adrs/ADR008-history-posture-per-record-type.md).

**Notes are distinct from overrides and touches; the flag names
say so.** Overrides correct closed-vocabulary judgments
(`override --reason <text>` for the justification); touches log
contact events (`touch --message <text>` for what was
communicated); contacts carry a stable description (`contact
--about <text>`); only `bdog note` produces a first-class C012
note record (`note add --text <text>`, with an auto-captured
context snapshot). Re-using `--note` across all four would
collapse four distinct concepts — a per-write reason, a
communication payload, a stable description, and a first-class
triage record — into one word. The flag names keep the four
distinguishable at the CLI surface where the bird-dogger
encounters them. `item add --first-note` is the one convenience
shortcut, named to make clear it constructs a `bdognote-N` not
an embedded field.

**Tokens never appear as CLI flags.** Tokens passed as flags
appear in shell history and process lists. `source add` (for
token-bearing types) and `source rotate-token` always prompt
interactively. `BDOG_SOURCE_<NAME>_TOKEN` provides the headless
path for CI and scripted environments. See
[ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md).

**Side-effecting actions live on their own verbs, not as flags
on `set`.** `source rotate-token`, `hunt activate`,
`hunt deactivate`, `item accept`, `item reject`,
`item carry-forward`, `item confirm-owner`, and `touch
record-response` are all distinct operations — recording an
event, advancing a state machine, or producing a new timestamp
— rather than field updates. Keeping them off `set` avoids the
value-less flag anti-pattern, keeps `set`'s semantics narrowly
field-update, and gives each action a clear failure surface
(`item accept` errors with `already in active set`; `set` has no
equivalent state-machine error to report). See
[ADR009](../adrs/ADR009-side-effecting-actions-on-dedicated-verbs.md).

**Active / inactive hunts.** Hunts carry an active flag;
default-arg `hunt bugel` operates on active hunts only.
Inactive hunts preserve their declaration and history without
running each cadence; explicit naming still operates on them,
and `--all` includes them when running without explicit names.
`bdog hunt activate` / `deactivate` flip the flag idempotently;
end-of-quarter cleanup deactivates rather than removes.

**`bugel` is the only refresh path.** The bugel sequences
refresh → re-assess → synthesize → render in one command,
matching the product's recurring-checkpoint framing. There is no
standalone `bdog hunt refresh` — partial work is covered by
`bdog hunt bugel --no-refresh` (skip the pull, re-synthesize
current State) and `--dry-run` (preview without writing). Per
[ADR002](../adrs/ADR002-on-demand-cadence-no-daemon.md), all
refresh / assessment / synthesis runs inside a single CLI
invocation; no background process. "Bugel" is a colloquialism
for the checkpoint concept — the recurring moment the bird-dogger
reviews coverage — and embraces the chase metaphor at the CLI
surface; the product-level docs use the formal "checkpoint." The
two terms refer to the same thing. Living under `hunt` keeps the
noun-first pattern intact and makes the hunt-scoped default
natural; `bdog hunt bugel` reads as "blow the bugel for this
hunt."

**Renderer surfaces are part of the design, not implementation
detail.** C015 carries PRD001 / PRD002 / PRD004 readability
obligations at the rendering layer; C011 carries the
list-level signal-selection obligation
(J001-O004-R002) on its synthesis output. Per-verb output
shapes are specified above so uncertainty states,
two-timestamp freshness, and override disagreement have concrete
columns and markers — not "we'll figure out the display later."
