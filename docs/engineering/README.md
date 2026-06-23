# Birddog

Birddog is a single Go binary (`bdog`) that runs the bird-dogging chase loop on the bird-dogger's local machine. It reads from authenticated external sources (Jira via `jira-cloud` and `jira-dc` source types; `local-json` files as a third first-class type), assesses items pulled into a hunt's coverage, and renders the resulting list at each on-demand checkpoint via `bdog hunt bugel` — `bugel` is the CLI's chase-metaphor name for the product's checkpoint moment; [CLI (C001)](components/C001-cli.md) carries the full rationale. All persistent runtime state — items, assessments, overrides, notes, the touch log — lives in one local SQLite database; declared sources, selectors, hunts, contacts, and chains live in one TOML config file; source secrets live in the OS keyring. The bird-dogger drives every state change through CLI subcommands; no daemon, no network listener, no server.

The architecture splits into four bands:

1. **Edge.** CLI (C001), Config Store (C002), Credential Store (C003), Renderer (C015). Everything the bird-dogger sees and types passes through these.
2. **Connection.** Source Registry (C004), Source Adapter (C005), Selector Registry (C017), Hunt Registry (C006). The two-axis decoupling — auth per source, scope per hunt — lives here; selectors are the shared scope primitive hunts compose.
3. **State.** Item Store (C007), Override Store (C010), Note Store (C012), Touch Log (C013), Contact Registry (C014), Coverage Memory (C016). The bird-dogger's facts about each item and the chase, kept locally. Per-item facts live in C007; per-hunt coverage state lives in C016.
4. **Reasoning.** Refresh Engine (C008), Assessment Engine (C009), Synthesis Engine (C011). The product's per-item readings and list-level recommendations, all derived from the State band on demand.

Reasoning reads from Connection and State, writes State, and hands results to the Renderer. It never writes back to sources.

## Principles

- **Coverage over precision.** Every per-item reading or produced judgment carries explicit "we don't know," "cannot assess," "cannot recommend," and "evidence disagrees" room rather than collapsing to a confident default. C009 (assessment readings), C011 (recommendations), and C014 (contact disagreement) hold this room first-class; the State-band stores that record those outputs preserve it. Realizes [PRD001](../product/pdrs/PRD001-coverage-over-precision.md).
- **One freshness contract.** Per-item state carries two timestamps — last-assessed and last-underlying-data-change — kept distinct; the active set carries a separate set-level last-refresh marker. Realizes [PRD002](../product/pdrs/PRD002-per-item-freshness-presentation.md) for the per-item shape, and extends it with a separate set-level marker for active-set freshness — the cousin shape PRD002 acknowledges but does not itself commit to.
- **One Source Registration capability.** The duplicate-at-the-requirement-layer source registration (J001-O001-R007 and J001-O002-R007) is realized by a single component bundle (C002 declaration, C003 secret, C004 registry, C005 adapter contract), not duplicated per outcome. Realizes [PRD003](../product/pdrs/PRD003-shared-source-registration.md).
- **One persistent override mechanism.** Status, intervention recommendation, and escalation timing recommendation all use the same Override Store, parameterized by which produced judgment the override targets. Realizes [PRD004](../product/pdrs/PRD004-override-surfacing.md).
- **Auth decoupled from scope.** A Source authenticates a connection to one external instance; a Selector names a reusable query against one source; a Hunt scopes a coverage set by referencing one or more selectors. The same source can back many selectors; the same selector can back many hunts; one hunt can mix selectors across sources. See [ADR001](adrs/ADR001-source-hunt-decoupling.md) (two-axis decoupling) and [ADR004](adrs/ADR004-selectors-as-first-class-declarations.md) (selectors as first-class declarations).
- **CLI-first, one-shot subcommands.** Every action exists as a composable, scriptable subcommand. List-level rendering is tabular text. An interactive TUI is out of scope for v1; revisit per ADR if `bdog hunt bugel` rendering proves cramped.
- **On-demand cadence, no daemon.** Refresh, re-assessment, and synthesis run inside a `bdog hunt bugel` (or a scoped equivalent) invocation. Nothing runs in the background.
- **Read-only against sources.** Source Adapters only read. Status overrides, notes, escalations, and touches are recorded locally; the bird-dogger sends the actual escalation out-of-tool. See [ADR003](adrs/ADR003-read-only-against-sources.md).
- **Local-only state.** One SQLite database, one TOML config, one keyring scope, all on the bird-dogger's host. No server, no network listener, no shared state.

## Constraints

- Single user per host. No multi-tenant model.
- Single static binary distributable via `go install` and a Homebrew tap (later). No installer, no separate runtime.
- All persistent runtime state in one SQLite database. All declarative configuration in one TOML file. All secrets in the OS keyring with a `BDOG_SOURCE_<NAME>_TOKEN` env-var fallback. Source types that need no secret (e.g., `local-json`) skip the keyring entirely; see [ADR005](adrs/ADR005-local-json-source-type.md).
- XDG Base Directory layout on Linux, OS-equivalent paths on macOS and Windows: config in `${XDG_CONFIG_HOME}/bdog/`, data in `${XDG_DATA_HOME}/bdog/`.
- Three source types are first-class: `jira-cloud` (Atlassian-hosted Jira), `jira-dc` (self-hosted Jira Data Center), and `local-json` (file-backed). Cloud and DC are distinct types because they differ in auth, endpoints, and rate-limit model despite sharing query semantics; `local-json` exercises the source-type-agnostic contract claim (see [ADR005](adrs/ADR005-local-json-source-type.md)). The Source Adapter contract must stay source-type-agnostic so future source types (other trackers, doc systems, message platforms) plug in without changes to the Reasoning band.
- Configuration is mutable from the CLI and from a text editor. Tokens are the single exception — they only flow through the CLI to the keyring, never through the TOML file.

## Technology Choices

- **Go**: single static binary, easy cross-platform shipping, strong CLI ecosystem, no runtime dependencies for the bird-dogger to install.
- **SQLite**: indices for ranking and freshness queries, ACID for the state band, single-file footprint that fits the local-only constraint. Driver selection (pure-Go vs. CGO) is ADR territory.
- **TOML**: human-editable, less ambiguous than YAML, good Go ecosystem support; serves the both-paths-mutable configuration goal.
- **OS keyring** (macOS Keychain, Linux Secret Service, Windows Credential Manager): native secret storage so source tokens never hit disk in plaintext. Library selection is ADR territory.
- **Jira API client**: source-type-specific; ADR territory. The Source Adapter contract isolates the choice from the rest of the system.

## Components

### C001 - CLI

Parses subcommands and arguments, dispatches to the relevant State or Reasoning component, and pipes results to the Renderer. Owns the `bdog` binary entry point; [CLI (C001)](components/C001-cli.md) is the sole owner of the full verb surface. Configuration nouns — source, selector, hunt, contact, chain — each support declaration and lifecycle management; chase nouns — item, override, touch, note — support per-item state management. The bugel subcommand sequences refresh → re-assess → synthesize → render for one or more hunts and is the only refresh path; a no-refresh variant re-synthesizes against current State without pulling sources. Tokens entered through the CLI flow to C003, never to C002.

#### Relationships

- **C002**: reads declared sources, selectors, hunts, contacts, and chains; writes config edits.
- **C003**: writes new tokens during source credential setup.
- **C004, C006, C014, C017**: writes registrations and edits.
- **C007, C016**: writes manual item captures, expected trajectories, and inter-item dependencies.
- **C008, C009, C010, C011, C012, C013**: dispatches chase actions.
- **C015**: hands results off for rendering.

### C002 - Config Store

Reads and writes the single TOML configuration file. Holds declarations as sibling tables: sources (`type`, `url` or `path`, non-secret auth fields, secret reference where applicable), selectors (`source`, `selector-type`, `value`), hunts (selector references, active flag), contacts (`name`, channel handles), and chains (ordered contact references). Re-read fresh on every CLI invocation. Holds no secrets.

Each declared entity carries a stable internal `id` field alongside its bird-dogger-chosen name, of shape `bdog<kind>-<n>` where `<kind>` is one of `source`, `selector`, `hunt`, `contact`, or `chain` and `<n>` is a monotonically increasing integer scoped per kind. Cross-entity references inside the TOML key on the id, not the name — a hunt's selector list, a chain's contact members, and an item's chain or owner reference all carry ids. This keeps `--rename` local: a name change rewrites the entity's name field without touching any reference. The CLI is the canonical id assigner; [CLI (C001)](components/C001-cli.md#behavior) covers the hand-edit reconciliation rule for new entities a bird-dogger adds by editing the TOML directly.

#### Relationships

- **C001**: read by every command; written by config CRUD verbs.
- **C004, C006, C014, C017**: source the source, hunt, contact / chain, and selector declarations respectively.

### C003 - Credential Store

Resolves a source secret via the OS keyring first, then the `BDOG_SOURCE_<NAME>_TOKEN` env-var fallback. Owns keyring read/write paths for source tokens. Tokens are only written via interactive prompt or the env-var — no CLI flag carries a secret string into C003. See [ADR010](adrs/ADR010-tokens-never-via-cli-flag.md). Asked only for sources whose type declares a secret; tokenless types (e.g., `local-json` per [ADR005](adrs/ADR005-local-json-source-type.md)) skip C003 entirely. The auth method (Basic, Bearer, etc.) is the Source Adapter's concern; C003 only stores and returns the secret string.

#### Relationships

- **C001**: writes new tokens on `source add` (when the type carries a secret) and on `source rotate-token`.
- **C005**: reads tokens at request time, scoped to one source.

### C004 - Source Registry

Holds the declared set of sources and resolves a source name to a `(declaration, secret, adapter)` bundle on demand. Validates source declarations on edit and on read.

#### Relationships

- **C001**: written by `source` declaration verbs (add, edit, remove); validates and delegates persistence to C002.
- **C002**: source declarations live here.
- **C003**: secrets are looked up here.
- **C005**: provides adapter instances bound to a source.
- **C006**: hunts reference sources by name, resolved here.

### C005 - Source Adapter

Source-type-specific contract for reading items from one external source and reporting source availability. Three source types realize the contract: `jira-cloud` (Atlassian-hosted Jira), `jira-dc` (self-hosted Jira Data Center), and `local-json` (file-backed). Cloud and DC are distinct types because they differ in auth shape, REST endpoints, and rate-limit model despite sharing query semantics; the C005 contract stays one, the realizations differ. [ADR005](adrs/ADR005-local-json-source-type.md) commits to `local-json` as the second concrete adapter exercising the source-type-agnostic claim. Future source types (other trackers, doc systems, message platforms) plug in the same way. Read-only per [ADR003](adrs/ADR003-read-only-against-sources.md).

#### Relationships

- **C003**: reads the bound source's secret per request, when the source's type carries one.
- **C004**: instantiated against a registered source.
- **C008**: invoked by the Refresh Engine to pull items and report availability.

### C006 - Hunt Registry

Holds the declared hunts. Each hunt is a named set of selector references resolved via C017, plus an active flag. The selector set is unordered: the refresh stage walks all referenced selectors per bugel without bird-dogger-controlled ordering. Active hunts run at each checkpoint; inactive hunts are skipped by default but can be included on demand. Inactive status preserves the hunt's declaration and history without re-running it each cadence. Validates that referenced selectors exist in C017.

#### Relationships

- **C001**: written by hunt declarations and lifecycle management (activation, deactivation, and selector membership edits); validates and delegates persistence to C002.
- **C002**: hunt declarations and selector references live here.
- **C017**: validates selector references; resolves selectors at refresh time.
- **C008**: receives the hunt's resolved selector set at refresh time.

### C007 - Item Store

Owns per-item facts: stable internal id of shape `bdogitem-<n>` per [ADR006](adrs/ADR006-native-id-scheme.md) (distinct from source ids), origin tag (manual vs. source-derived), recorded expected trajectories, and inter-item dependencies. Also holds the per-item assessment readings and basis traces C009 writes back, and the per-item underlying-data-change timestamp C008 writes on refresh (the second of PRD002's two distinct timestamps; "last assessed" lives on the assessment reading C009 writes). Identity hub for the State band — every other State-band store (C010, C012, C013, C014, C016) keys into items by C007's internal id. SQLite-backed.

#### Relationships

- **C001**: written by chase verbs that record per-item facts (manual item capture, expected-trajectory recording, inter-item dependency recording).
- **C008**: writes new items and starting expected trajectories from source-side due dates.
- **C009**: read for assessment; written with assessment readings and basis traces.
- **C010, C012, C013, C014, C016**: hold their own per-item or per-hunt state but reference items by C007's internal id.
- **C011**: read at synthesis time.
- **C015**: read at render time.

### C008 - Refresh Engine

For a target hunt, walks each referenced selector via C017, resolves it to a `(source, selector-type, value)` triple, pulls items via C005, writes new items to C007, writes the per-item underlying-data-change timestamp to C007, and writes per-hunt coverage state (active set, source-availability, set-level refresh marker) to C016. Adopts source-side due dates as starting expected trajectories on C007. The multi-source picking rule for the underlying-data-change timestamp (max-of, per-source array, composite) is ADR territory per PRD002; C008 holds the write site once the rule lands. Triggered on-demand by `bdog hunt bugel`; `bdog hunt bugel --no-refresh` skips this stage and re-synthesizes against current State without invoking C008.

#### Relationships

- **C005**: reads items and availability.
- **C006**: receives the target hunt and its selector references.
- **C017**: resolves selector references to executable triples.
- **C007**: writes new items, starting expected trajectories, and the per-item underlying-data-change timestamp.
- **C016**: writes per-hunt active set, source-availability, and the set-level refresh marker.
- **C009**: invokes re-assessment on items whose underlying data has changed since their last assessment.

### C009 - Assessment Engine

Computes the per-item slip reading (trajectory comparison, silence, combined slip status) and the per-item status reading (basis, confidence, conflict, freshness) over the active set in C016, drawing per-item facts from C007. Carries "cannot assess," "cannot determine confidence," and "sources disagree" states explicitly per Coverage-over-Precision. Writes assessment readings and basis traces back to C007.

#### Relationships

- **C007**: reads per-item facts and underlying-data signals; writes assessment readings.
- **C016**: reads the per-hunt active set the assessment runs against.
- **C008**: invoked on items flagged as changed since last assessment.
- **C010**: an active override is surfaced alongside the inferred reading; assessment never silently overwrites it.
- **C011**: produces the inputs synthesis consumes.

### C010 - Override Store

One persistent override mechanism, parameterized by the produced judgment it covers: status (from C009), intervention recommendation (from C011), and escalation timing recommendation (from C011). Each override survives re-synthesis; new produced output is surfaced alongside the active override for the bird-dogger to adopt or dismiss. Cleared overrides remain recoverable in history.

#### Relationships

- **C001**: written by override declarations targeting produced judgments.
- **C009, C011**: read at render time so the override stands as the displayed value with disagreement surfaced alongside.
- **C015**: receives override + produced output as a pair.

### C011 - Synthesis Engine

Reads C007 (per-item facts), C016 (per-hunt coverage state), C009 (assessment), C010 (overrides), C012 (prior notes), C013 (touch / escalation history), and C014 (owner and chain) to produce per-item urgency ranking, intervention recommendation, and escalation timing recommendation. Carries "cannot recommend" explicitly when inputs are insufficient, per Coverage-over-Precision composed with the override mechanism (PRD001 + PRD004). Also emits a per-item list-level signal pack — which urgency-driving facts to surface alongside each row, ranked by salience to that item's urgency — per J001-O004-R002. The signals themselves are produced under O001 (slip / assessment) and O003 (status); selecting which appear at the list level is a synthesis decision, not a rendering one. Writes nothing back to its inputs; emits a synthesis result consumed by C015.

#### Relationships

- **C007, C009, C010, C012, C013, C014, C016**: read at synthesis time.
- **C015**: receives ranking and recommendations.

### C012 - Note Store

Records per-item open-vocabulary notes. Each note carries free text supplied by the bird-dogger plus a snapshot of context at write time (rank, signals, recommendation, active overrides) so later readers see both what was decided and what was visible when the decision was made. Notes are the bird-dogger's working thoughts during a checkpoint — distinct from `C010` overrides (closed-vocabulary judgment corrections) and from `C013` touches (contact events). Realizes `J001-O004-R004 Triage State Carry-Forward`: at the next checkpoint, prior notes surface alongside the current readings with the two-timestamp freshness shape applied. Hard-deletable: notes correct a recording mistake the same way touches do. SQLite-backed.

#### Relationships

- **C001**: written by note declarations and management.
- **C007**: references items by stable internal id.
- **C009, C010, C011**: read at write time to capture the context snapshot (rank, signals, active overrides, recommendation).
- **C011**: read as a synthesis input — prior notes inform recommendation framing.
- **C015**: read at render time for the prior-notes surface in item info and bugel.

### C013 - Touch Log

Append-only log of bird-dogger touches on items, including escalation attempts: target (a C014 contact, typically resolved through a chain member), channel, time, response state, and a `type` field distinguishing touch kinds. The record shape is shared across all touch types. The per-item response window (now − most-recent-touch time) is derived by readers from the log, not stored; C013 anchors the derivation by holding the touch times.

#### Relationships

- **C001**: written by touch declarations; the `type` field records the touch kind at write time.
- **C007, C014**: references items by stable id and contacts by name.
- **C011**: read as a synthesis input for timing recommendation (prior attempts and the derived response window).
- **C015**: read at render time for touch history and the derived response window.

### C014 - Contact Registry

Two layers. The declarative layer holds bird-dogger-registered contacts (people, with channel handles) and named ordered chains that reference them; both are top-level entities the bird-dogger maintains via contact and chain declarations (persisted as sibling tables in C002). The state layer holds per-item owner references with owner-confirmation freshness and per-item source-derived contact data; disagreements between bird-dogger-registered owners and source-derived owners are surfaced rather than collapsed (a PRD001 application). Items reference one contact for ownership and optionally one chain for escalation; both references resolve through this component.

Living in the State band — despite the contact/chain declarations being declarative — keeps the per-item owner state and disagreement surfacing co-located with the declarations they're built on. The declarative layer is the State-band's counterpart to source/selector/hunt registries in Connection.

#### Relationships

- **C001**: written by contact and chain declarations, by chain membership edits, by item-owner assignment and confirmation, and by item-chain assignment. Also written when source-derived owners enter via the adapter.
- **C002**: contact and chain declarations live here.
- **C007**: per-item references resolve to C007's internal item ids; contacts and chains are referenced by name.
- **C011**: read at synthesis time for escalation timing and owner-currency input.
- **C015**: read at render time for owner and chain display, and for the registered-vs-source-derived disagreement surface.

### C015 - Renderer

Produces tabular CLI output for list-level views and item-level detail views; per-verb output shapes live in [Renderer (C015)](components/C015-renderer.md#renderer-surfaces). Carries the readability obligations from PRD001 (uncertainty / disagreement states stay readable, not buried) and PRD002 (two timestamps surfaced distinctly per item) at the rendering layer — column layout, density given table width, overflow behavior, marker glyphs. Renders the per-item signal pack C011 emits; does not pick which signals matter (that's synthesis). Writes only to stdout/stderr; holds no state.

#### Relationships

- **C007, C009, C010, C011, C012, C013, C014, C016**: read for the data to render.
- **C001**: invoked by the CLI to produce output.

### C016 - Coverage Memory

Owns per-hunt coverage state: active set membership per hunt, source-availability state per hunt-refresh, and the set-level last-refresh marker. Keyed by `(hunt, item)` or `(hunt, source)` pairs; items are referenced by C007's internal id. SQLite-backed.

**Coverage scope rule.** For each hunt, the active set is the union of all items returned by the hunt's selectors on last refresh and manually captured items assigned to that hunt. Scope is adjusted by editing selectors (`selector`, `hunt selector` verbs) or adding/removing manual items (`item add`, `item remove`). There is no per-item accept/reject for source-backed items; when an item stops matching a hunt's selectors, it leaves the active set on the next refresh with no separate drop-off surface.

The per-item / per-hunt seam: C007 holds facts about items regardless of hunt; C016 holds answers to "what does this hunt currently include and last refreshed when."

#### Relationships

- **C001**: written by chase verbs that change coverage state (manual item capture).
- **C007**: references items by internal id.
- **C008**: written on every refresh.
- **C009**: read at assessment time (active set scopes assessment).
- **C011**: read at synthesis time.
- **C015**: read at render time.

### C017 - Selector Registry

Holds the declared set of selectors per [ADR004](adrs/ADR004-selectors-as-first-class-declarations.md). Each selector is a named `(source, selector-type, value)` triple. Resolves a selector name to its triple on demand and validates that the referenced source exists in C004. The same selector can back many hunts; editing a selector propagates to every referencing hunt on the next refresh, satisfying the share-at-component-layer posture PRD003 sets for the auth axis and ADR004 extends to the scope axis.

#### Relationships

- **C001**: written by selector declarations and tested on demand; validates and delegates persistence to C002.
- **C002**: selector declarations live here.
- **C004**: validates source references.
- **C006**: hunts reference selectors by name; C006 validates references against C017.
- **C008**: resolves selector names to executable triples at refresh time.

## Requirement-Component Map

Every requirement is exposed via **C001 (CLI)** and rendered via **C015 (Renderer)**. The map below names the additional components that satisfy each requirement.

- **J001-O001-R001** — Proactive Slip Surfacing: C009, C008
- **J001-O001-R002** — Assessment Freshness: C009, C007
- **J001-O001-R003** — Expected Trajectory: C007, C008
- **J001-O001-R004** — Trajectory Comparison: C009
- **J001-O001-R005** — Silence As Signal: C009
- **J001-O001-R006** — Source Availability: C005, C008, C016
- **J001-O001-R007** — Source Registration: C004, C005, C003, C002
- **J001-O002-R001** — Coverage Set Visibility: C016, C007, C004, C017, C006
- **J001-O002-R002** — Manual Item Capture: C007, C016
- **J001-O002-R006** — Active Set Freshness: C008, C016
- **J001-O002-R007** — Source Registration: C004, C005, C003, C002
- **J001-O003-R001** — Status Basis: C009, C007
- **J001-O003-R002** — Status Freshness: C009, C007
- **J001-O003-R003** — Status Confidence: C009
- **J001-O003-R004** — Status Override: C010
- **J001-O003-R005** — Re-Assessment On Change: C008, C009
- **J001-O003-R006** — Conflict Surfacing: C009
- **J001-O004-R001** — Urgency Ranking: C011
- **J001-O004-R002** — List-Level Urgency Signals: C011 (list-level signal selection — which signals surface per row at the list view; the signals themselves are produced under O001 and O003 and rendered by C015)
- **J001-O004-R003** — Intervention Recommendation: C011, C010
- **J001-O004-R004** — Triage State Carry-Forward: C012 (Note Store: prior notes plus snapshot surface at the next checkpoint)
- **J001-O004-R005** — Inter-Item Dependency Surfacing: C007
- **J001-O005-R001** — Escalation Target: C014, C007
- **J001-O005-R002** — Owner Currency: C014
- **J001-O005-R003** — Response Window Visibility: C013
- **J001-O005-R004** — Escalation Timing Recommendation: C011, C010
- **J001-O005-R005** — Escalation Record: C013
- **J001-O005-R006** — Escalation Contact Registration: C014
