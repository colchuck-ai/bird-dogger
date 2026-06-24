# CLI (C001)

Parses the `bdog` command surface, validates arguments, resolves names and ids, dispatches to State and Reasoning components, and hands results to C015 for rendering. Owns the binary entry point. Every bird-dogger action is a composable, scriptable subcommand; there is no daemon and no background process.

## Data model

**Binary and product names.** The product is Birddog. The binary is `bdog`.

**Native ids.** Every auto-assigned entity uses `bdog<kind>-<n>`: `<kind>` is the entity name, `<n>` is a monotonically increasing integer scoped per kind. Examples: `bdogitem-123`, `bdognote-7`, `bdogtouch-87`. Declarative entities (`source`, `selector`, `hunt`, `contact`, `chain`) also carry stable internal ids assigned on `add`, in addition to bird-dogger-chosen names.

**Storage keys on ids; CLI speaks names.** Cross-entity references in the TOML config (C002) and SQLite stores key on id, not name. The CLI resolves `<name>` to id at command time. Renames via `--rename` rewrite the name field only; references stay pointed at the same id. `bdog <noun> info` and `bdog <noun> list` surface the id alongside the name so hand-editing the TOML can match either form. Reusing a freshly-vacated name for a new entity is safe — old references continue to point at the renamed predecessor's id, not at the new entity.

**Item identity.** Every item gets a `bdogitem-<n>` on first capture (manual) or first refresh (source-derived). Source-derived items retain their source id (e.g. `SUPPLY-1234`) as an alias. Commands accepting `<item>` accept either form:
- Native id — always unambiguous.
- Source id — works when it resolves to exactly one item in the State band. Ambiguous matches fail fast with the colliding items listed; the bird-dogger must use native ids.

Overrides and chain members use composite keys; they have no standalone id.

**Positional argument shapes by noun kind.**
- Declarative nouns (`source`, `selector`, `hunt`, `contact`, `chain`): bird-dogger-chosen name as positional on `add`.
- Chase-state nouns (`item`, `touch`, `note`): auto-assigned ids on `add`; positional is parent context where applicable (`bdog touch add <item>`), not a chosen name. The assigned id is printed.
- Composite-key resources: identity pair as positionals (`bdog chain member add <chain> <contact>`, `bdog override add <item> <field> <value>`).

## Interfaces

### Command pattern

```
bdog <noun> <sub-verb> [<args>] [<flags>]
```

Standard sub-verbs: `add | remove | info | list | set`. Every top-level noun exposes all five unless carved out.

**Carve-outs.**
- Sub-resources with no state of their own (`hunt selector`, `item depends-on`): `add | remove | list` only.
- Action verbs that are not field updates: documented per noun below.

**Global flags** (every command):

```
--output, -o  text|json    default: text
--quiet,  -q               suppress headers and labels; data only
```

**Add vs set.** `add` errors on existing; `set` errors on missing. No silent upsert. Applies uniformly, including composite-key resources.

**Field clearing.**
- Typed reference fields: sentinel `none` (`bdog item set <item> --chain none`, `--owner none`). `none` is reserved as an entity name.
- Free-text fields: `--clear-<field>` (`bdog contact set alice --clear-email`). Value-bearing and `--clear-*` flags are mutually exclusive in one invocation.

**Timestamps.** Inputs (`--at`, `--responded-at`, `--due`, etc.) parse as ISO 8601 in the host local zone unless the value carries an explicit offset. Date-only: `YYYY-MM-DD`. Datetime: `YYYY-MM-DDTHH:MM[:SS][offset]`.

### source

Connection to one external system instance. Secrets never appear as CLI flags; token-bearing types prompt on `add` and `rotate-token`. Headless fallback: `BDOG_SOURCE_<NAME>_TOKEN` env var (`ttd-jira` → `BDOG_SOURCE_TTD_JIRA_TOKEN`).

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
    --verbose

bdog source rotate-token <name>
```

Type is fixed at `add`. `set` rejects `--url` on `local-json` and `--path` on remote types. `remove` fails when any selector references the source.

### selector

Named, reusable query against one source. `--selector-type` required on `add` (`project|jql|filter`); not inferred from value shape.

```
bdog selector add <name>
    --source         <source-name>           required
    --selector       <value>                 required
    --selector-type  project|jql|filter      required

bdog selector remove <name>
bdog selector info   <name>
bdog selector list
    --source         <source-name>
    --selector-type  project|jql|filter

bdog selector set <name>
    --rename         <new-name>
    --source         <source-name>
    --selector       <value>
    --selector-type  project|jql|filter

bdog selector test <name>
    --verbose
```

`remove` fails when any hunt references the selector.

### contact

Declared person with channel handles and stable description.

```
bdog contact add <name>
    --email   <email>
    --slack   <handle>
    --jira    <username>
    --about   <text>

bdog contact remove <name>
bdog contact info   <name>
bdog contact list

bdog contact set <name>
    --rename        <new-name>
    --email         <email>
    --slack         <handle>
    --jira          <username>
    --about         <text>
    --clear-email | --clear-slack | --clear-jira | --clear-about
```

`remove` fails when referenced by any chain or as any item owner.

`--about` is a stable freetext description of the contact — distinct from `bdog note` records (item-scoped, auto-capture a context snapshot at write time), `bdog override --reason` (per-write justification for a judgment correction), and `bdog touch --message` (content of a contact event). Channel-handle flags and `--about` are free text; clearing uses `--clear-<field>`. `--clear-*` flags and their value-bearing siblings are mutually exclusive in one invocation (e.g., `--email foo@bar --clear-email` errors).

### chain

Named, reusable escalation contact list. Members ordered by position (1-indexed, contiguous). Position 1 is primary.

```
bdog chain add <name>
bdog chain remove <name>
bdog chain info   <name>
bdog chain list
bdog chain set <name>
    --rename  <new-name>
```

**chain member:**

```
bdog chain member add <chain> <contact>
    --position  <n>     insert at n; omit to append

bdog chain member remove <chain> <contact>
bdog chain member info <chain> <contact>
bdog chain member list <chain>
bdog chain member set <chain> <contact>
    --position  <n>
```

`add` inserts at the given position and shifts higher-numbered members down; omitting `--position` appends. `remove` closes the gap automatically. `set --position` moves a member and shifts others to maintain a contiguous list. `add` errors if the contact is already a member; `set` errors if not. Contact must exist in C014 first.

### hunt

Named collection of selector references; unit of bugel scoping. Carries active flag.

```
bdog hunt add <name>
bdog hunt remove <name>
bdog hunt info   <name>
bdog hunt list   [--all | --inactive]

bdog hunt set <name>
    --rename      <new-name>

bdog hunt activate   <name>
bdog hunt deactivate <name>

bdog hunt bugel [<hunt>...]
    --no-refresh
    --dry-run
    --all
    --verbose
```

No-arg `bugel` runs active hunts only. Explicit hunt names override the active filter. `remove` fails when manual items are assigned to the hunt.

**hunt selector** (unordered set membership):

```
bdog hunt selector add    <hunt> <selector>...
bdog hunt selector remove <hunt> <selector>...
bdog hunt selector list   <hunt>
```

`add` with varargs; errors if any selector already associated; partial state is not written.

### item

Tracked open work unit — source-derived or manually captured.

```
bdog item add
    --title       <text>      required
    --hunt        <hunt>      required
    --due         <date>
    --first-note  <text>

bdog item remove <item>       manual only
bdog item info <item>
    --show-basis | --show-history

bdog item list
    --hunt    <hunt>
    --sort    urgency|due|freshness|slip-age    default: urgency

bdog item set <item>
    --title | --trajectory due-date|pace|none | --due | --pace
    --hunt <hunt>               manual only
    --chain <chain>|none | --owner <contact>|none
    --clear-pace

bdog item confirm-owner <item>
```

`confirm-owner` records the owner-confirmation timestamp; each invocation advances it (not idempotent by design).

Trajectory cross-validation: `due-date` requires `--due`; `pace` requires `--pace`; `none` clears both. Conflicting trajectory and field combinations error with resolution hint.

**Source-backed item scope.** Source-backed items enter a hunt's active set when they match the hunt's selectors on refresh; they leave on the next refresh when they no longer match. Scope is adjusted by editing selectors (`hunt selector add/remove`, `selector set`). There is no per-item accept/reject for source-backed items; `item add` and `item remove` apply to manually captured items only.

**item depends-on:**

```
bdog item depends-on add    <item> <other-item>...
bdog item depends-on remove <item> <other-item>...
bdog item depends-on list   <item>
```

### override

Bird-dogger correction to a produced judgment. Identity: `(item, field)` positional pair. Fields: `status`, `intervention`, `escalation-timing`.

```
bdog override add <item> <field> <value>
    --reason  <text>

bdog override remove <item> <field>
bdog override info <item> [<field>]
bdog override list
    --item | --field | --all

bdog override set <item> <field> <value>
    --reason  <text>
```

Value vocabularies: `intervention` → `intervene-now|not-now|cannot-recommend`; `escalation-timing` → `escalate-now|not-yet|cannot-recommend`; `status` → source-type vocabulary or freeform for manual items.

### touch

Recorded contact event. Response window derived at read time, not stored.

```
bdog touch add <item>
    --type      escalation|checkin            required
    --channel   <channel>                     required
    --target    <contact>
    --message   <text>
    --at        <datetime>                    default: now

bdog touch remove | info <id>
bdog touch list
    --item | --since | --channel | --type | --open

bdog touch set <id>
    --channel | --target | --at | --type | --message

bdog touch record-response <id>
    --response       <text>                   required
    --responded-at   <datetime>               required
```

### note

Per-item open-vocabulary annotation with auto-captured context snapshot at write time.

```
bdog note add <item>
    --text  <text>     required

bdog note remove | info <id>
bdog note list
    --item | --since

bdog note set <id>
    --text  <text>     required
```

### Output contract

List-level and detail output shapes are specified in [Renderer (C015)](C015-renderer.md). C001 invokes C015 after dispatch; it does not format tabular output itself.

## Behavior

**Hand-edit reconciliation (TOML).** The config file is mutable from CLI and text editor. New entities added by hand omit `id`; the next CLI read assigns `bdog<kind>-<n>`, writes it back, and emits a one-line acknowledgement. Cross-references in TOML must use id form, not name. Hand-written ids that collide or skip sequence cause fail-fast on next CLI invocation.

**Removal semantics.**
- Hard delete: `touch remove`, `note remove`.
- Deactivate, preserve history: `override remove` (per PRD004); `override list --all` shows deactivated history.
- No-cascade for declarative entities and manual items: `remove` checks outstanding references and fails with blocking references and clearing verbs.

| Removing | Required pre-step |
|---|---|
| `source` | Remove selectors referencing it |
| `selector` | Remove from every hunt (`hunt selector remove`) |
| `contact` | Remove from chains; clear item owners |
| `chain` | Clear from every item (`item set --chain none`) |
| `hunt` | Reassign or remove manual items |
| `item` (manual) | None |

**Idempotence.**

| Verb shape | Re-invocation behavior |
|---|---|
| `add` | Errors on duplicate identity |
| `set` | Idempotent on identical input |
| `remove` (most entities) | Errors if not found |
| `override remove` | Idempotent; deactivates or no-op |
| `info`, `list`, `*-test` | Pure reads |
| `hunt bugel` | Re-runs full pipeline; timestamps advance |
| `hunt bugel --no-refresh` | Idempotent on identical State-band input |
| `touch record-response` | Idempotent on identical input; overwrites on different input |
| `hunt activate/deactivate` | Idempotent |
| `item confirm-owner` | Not idempotent; advances timestamp each invocation |

**Bugel as sole refresh path.** `hunt bugel` sequences refresh → re-assess → synthesize → render. No standalone refresh command. `--no-refresh` skips the pull; `--dry-run` previews without writing.

**Side-effecting actions on own verbs.** `source rotate-token`, `hunt activate`/`deactivate`, `item confirm-owner`, and `touch record-response` are not flags on `set` — they record events or advance state machines.

**Text field naming.** `--reason` (override), `--message` (touch), `--about` (contact), `--text` (note) keep four distinct concepts separate at the CLI surface.

## Edge cases

- **Ambiguous source id:** fail with matching items listed; use native id.
- **Trajectory conflicts:** setting `--due` on `pace` trajectory (or vice versa) errors with fix hint.
- **Token security:** tokens never as flags; interactive prompt or env var only.
- **Filter flags on `list`:** single value per flag; broaden by re-running or `--all` where defined.
- **Unregistered touch target:** add contact first, or omit `--target` and capture person in `--message`.
- **Source-derived item removal:** rejected with pointer to owning selector.

## Relationships

- **C002**: reads and writes declared sources, selectors, hunts, contacts, chains; canonical id assigner for config entities.
- **C003**: writes tokens on `source add` and `source rotate-token` (never to C002).
- **C004, C006, C014, C017**: dispatches registration and edit verbs.
- **C007, C016**: dispatches manual item capture, trajectories, and dependencies.
- **C008, C009, C010, C011, C012, C013**: dispatches chase actions including `hunt bugel`.
- **C015**: hands structured results for rendering.

## Success criteria

- Every product requirement exposed via a documented subcommand (see architecture requirement–component map).
- Noun-first sub-verb pattern is consistent across all managed entities; tab completion surfaces a uniform shape.
- No silent upsert, no cascade delete, no ambiguous item resolution.
- Script-safe idempotent verbs (`info`, `list`, `activate`/`deactivate`, identical `set`) behave predictably under cron/launchd/systemd per ADR002.
- Hand-edited TOML and CLI paths reconcile without reference corruption.

## Notes

Supersedes the placeholder verb list previously in the engineering root README.

### Design decisions

**Noun-first sub-verb pattern.** Every managed entity follows `bdog <noun> <sub-verb>`. Top-level nouns expose the full `add|remove|info|list|set` set; stateless sub-resources (`hunt selector`, `item depends-on`) expose `add|remove|list` only. Learning one noun transfers to all others; tab completion surfaces a consistent shape across the entire binary.

**Storage keys on stable ids; the CLI speaks names.** Every declared entity carries a `bdog<kind>-<n>` id alongside its bird-dogger-chosen name. Cross-entity references in the TOML config and SQLite stores key on the id, not the name. Renames are therefore local — there is no propagation step and no risk of a half-updated reference. See ADR006 for the native-id scheme and source-id alias design.

**Selectors are top-level, not hunt-owned.** A selector's definition (source, query, type) is independent of which hunts use it. Per ADR004, lifting selectors to a first-class noun lets the same query appear in multiple hunts and lets changes propagate without per-hunt edits.

**Contacts are top-level; chains reference contacts.** A contact is the person; a chain is a named ordered list of contacts; an item references one contact for ownership and optionally one chain for escalation. Lifting contacts out of chains lets the same person appear in multiple chains and as multiple items' owner without re-declaration; channel handles, freshness, and disagreement state live with the contact.

**`hunt selector` exposes only `add|remove|list`.** The sub-resource records only membership — there is nothing to `set` or inspect via `info` that does not already live on the `selector` noun. The asymmetry with `chain member` is intentional: `chain member` carries its own state (position) and its order is bird-dogger-visible; `hunt selector` is an unordered set, and the refresh engine walks all referenced selectors per bugel without bird-dogger-controlled ordering.

**Manual items carry no source reference.** `item add` accepts no URL or source identifier. A manual item is a purely local record. Tracking a specific remote item that falls outside existing selectors is done by creating a targeted selector — this keeps item identity clean and avoids a hybrid manual-plus-remote category.

**No-cascade removal.** Removing a declarative entity (`source`, `selector`, `contact`, `chain`, `hunt`) or manual item is rejected when any consumer still references it. The error names the blocking references and the verbs that would clear them. Cascade would let one verb silently destroy chains of declarations and per-item history; failing fast keeps the bird-dogger in control. See ADR007.

**`override remove` deactivates; `touch remove` and `note remove` hard-delete.** Overrides are deactivated on remove — history is preserved per PRD004. Touches and notes are hard-deleted — they correct a recording mistake and carry no equivalent history obligation. See ADR008.

**Active/inactive hunts.** Hunts carry an active flag; default-arg `hunt bugel` operates on active hunts only. Inactive hunts preserve their declaration and history without running each cadence; explicit naming still operates on them. End-of-quarter cleanup deactivates rather than removes.

**`bugel` is the only refresh path.** The bugel sequences refresh → re-assess → synthesize → render in one command, matching the product's recurring-checkpoint framing. There is no standalone `bdog hunt refresh` — partial work is covered by `--no-refresh` (skip pull, re-synthesize current State) and `--dry-run` (preview without writing). Per ADR002, all refresh/assessment/synthesis runs inside a single CLI invocation; no background process. "Bugel" is a colloquialism for the checkpoint concept — product docs use the formal "checkpoint." The two terms refer to the same thing.

### Common workflows

Worked examples for bird-doggers and implementers. They illustrate composition of the verbs above; they are not additional requirements.

#### First-time setup

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

#### Daily bugel

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

#### Recording a touch and its response

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

#### Adding a note during triage

```sh
bdog note add bdogitem-1234 \
    --text "alice flagged this in standup; bob is the code-review blocker. \
            watching for friday landing."

# later: list this item's notes (newest first)
bdog note list --item bdogitem-1234
```

#### Overriding a status judgment

```sh
bdog item info bdogitem-1234 --show-basis

bdog override add bdogitem-1234 status blocked \
    --reason "alice confirmed in standup; jira status is stale"

# clear once the source catches up
bdog override remove bdogitem-1234 status
```

#### Setting up contacts, a chain, and escalating

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

#### Capturing a manual item

```sh
bdog item add \
    --title "partner API rate limit increase" \
    --hunt supply-q3 \
    --due 2026-07-15 \
    --first-note "committed in q3 planning; tracked by dave's team, not in jira"

# the item's id (bdogitem-N) is printed; use it on subsequent commands
bdog item set bdogitem-487 --trajectory due-date --due 2026-07-15 --owner dave
```

#### Adding a selector to a live hunt

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

# later: narrow scope by editing the selector — no per-item action
bdog selector set supply-incidents \
    --selector "project = SUPPLY AND issuetype = Incident AND priority = P0 AND status != Done"
bdog hunt bugel supply-q3 --dry-run
bdog hunt bugel supply-q3
# items that no longer match leave the active set on the next refresh
```

#### Reusing a selector across two hunts

```sh
bdog hunt add infra-q3
bdog hunt selector add infra-q3 supply-p0

# broaden scope; propagates to both hunts on next bugel
bdog selector set supply-p0 \
    --selector "project in (SUPPLY, INFRA) AND priority = P0 AND status != Done"
bdog hunt bugel --dry-run
bdog hunt bugel
```

#### Deactivating a hunt at end of quarter

```sh
bdog hunt deactivate supply-q3

# inactive hunts are skipped by default; explicit invocation still works
bdog hunt bugel supply-q3   # runs supply-q3 even though inactive

# later, reactivate
bdog hunt activate supply-q3
```

#### Rotating a source token

```sh
bdog source rotate-token ttd-jira
# interactive prompt for the new token; written to keyring
bdog source test ttd-jira
```

#### Tearing down a source (no-cascade)

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
