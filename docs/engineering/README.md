# Birddog

Birddog is a single Go binary CLI tool that runs the bird-dogging
chase loop on the bird-dogger's local machine. It reads from
authenticated external sources (Jira first), assesses items pulled
into a hunt's coverage, and renders the resulting list at each
on-demand checkpoint. All persistent runtime state — items,
assessments, overrides, triage decisions, the touch log — lives in
one local SQLite database; declared sources and hunts live in one
TOML config file; source secrets live in the OS keyring. The
bird-dogger drives every state change through CLI subcommands; no
daemon, no network listener, no server.

The architecture splits into four bands:

1. **Edge.** CLI (C001), Config Store (C002), Credential Store
   (C003), Renderer (C015). Everything the bird-dogger sees and
   types passes through these.
2. **Connection.** Source Registry (C004), Source Adapter (C005),
   Hunt Registry (C006). The two-axis decoupling — auth per
   source, scope per hunt — lives here.
3. **State.** Item Store (C007), Override Store (C010), Triage
   Memory (C012), Touch Log (C013), Contact Registry (C014). The
   bird-dogger's facts about each item and the chase, kept
   locally.
4. **Reasoning.** Refresh Engine (C008), Assessment Engine (C009),
   Synthesis Engine (C011). The product's per-item readings and
   list-level recommendations, all derived from the State band on
   demand.

Reasoning never writes back to sources; it reads State, writes
State, and hands results to the Renderer.

## Principles

- **Coverage over precision.** Every per-item state model carries
  explicit "we don't know," "cannot assess," and "evidence
  disagrees" room rather than collapsing to a confident default.
  Realizes
  [PRD001](../product/pdrs/PRD001-coverage-over-precision.md).
- **One freshness contract.** Per-item state carries two
  timestamps — last-assessed and last-underlying-data-change —
  kept distinct; the active set carries a separate set-level
  last-refresh marker. Realizes
  [PRD002](../product/pdrs/PRD002-per-item-freshness-presentation.md).
- **One Source Registration component.** The duplicate-at-the-
  requirement-layer source registration (J001-O001-R007 and
  J001-O002-R007) collapses to one Source Registry plus one
  Source Adapter contract. Realizes
  [PRD003](../product/pdrs/PRD003-shared-source-registration.md).
- **One persistent override mechanism.** Status, intervention
  recommendation, and escalation timing recommendation all use
  the same Override Store, parameterized by which produced
  judgment the override targets. Realizes
  [PRD004](../product/pdrs/PRD004-override-surfacing.md).
- **Auth decoupled from scope.** A Source authenticates a
  connection to one external instance; a Hunt scopes a coverage
  set via one or more `(source, selector)` entries. The same
  source can back many hunts; one hunt can mix sources.
- **CLI-first, one-shot subcommands.** Every action exists as a
  composable, scriptable subcommand. List-level rendering is
  tabular text. An interactive TUI is out of scope for v1;
  revisit per ADR if `birddog checkpoint` rendering proves
  cramped.
- **On-demand cadence, no daemon.** Refresh, re-assessment, and
  synthesis run inside a `birddog checkpoint` (or a scoped
  equivalent) invocation. Nothing runs in the background.
- **Read-only against sources.** Source Adapters only read.
  Status overrides, triage decisions, escalations, and touches
  are recorded locally; the bird-dogger sends the actual
  escalation out-of-tool.
- **Local-only state.** One SQLite database, one TOML config, one
  keyring scope, all on the bird-dogger's host. No server, no
  network listener, no shared state.

## Constraints

- Single user per host. No multi-tenant model.
- Single static binary distributable via `go install` and a
  Homebrew tap (later). No installer, no separate runtime.
- All persistent runtime state in one SQLite database. All
  declarative configuration in one TOML file. All secrets in the
  OS keyring with a `BIRDDOG_SOURCE_<NAME>_TOKEN` env-var
  fallback.
- XDG Base Directory layout on Linux, OS-equivalent paths on
  macOS and Windows: config in `${XDG_CONFIG_HOME}/birddog/`,
  data in `${XDG_DATA_HOME}/birddog/`.
- Read-only access to external sources. The product writes
  nothing back to Jira or any future source.
- Jira is the first source type. The Source Adapter contract
  must be source-type-agnostic so future source types (other
  trackers, doc systems, message platforms) plug in without
  changes to the Reasoning band.
- Configuration is mutable from the CLI and from a text editor.
  Tokens are the single exception — they only flow through the
  CLI to the keyring, never through the TOML file.

## Technology Choices

- **Go**: single static binary, easy cross-platform shipping,
  strong CLI ecosystem, no runtime dependencies for the
  bird-dogger to install.
- **SQLite**: indices for ranking and freshness queries, ACID
  for the state band, single-file footprint that fits the
  local-only constraint. Driver selection (pure-Go vs. CGO) is
  ADR territory.
- **TOML**: human-editable, less ambiguous than YAML, good Go
  ecosystem support; serves the both-paths-mutable configuration
  goal.
- **OS keyring** (macOS Keychain, Linux Secret Service, Windows
  Credential Manager): native secret storage so source tokens
  never hit disk in plaintext. Library selection is ADR
  territory.
- **Jira API client**: source-type-specific; ADR territory. The
  Source Adapter contract isolates the choice from the rest of
  the system.

## Components

### C001 - CLI

Parses subcommands and arguments, dispatches to the relevant
State or Reasoning component, and pipes results to the Renderer.
Owns configuration CRUD verbs (`source`, `hunt`, `contact`) and
chase verbs (`checkpoint`, `triage`, `status`, `recommend`,
`escalate`, `touch`). Tokens entered through the CLI flow to
C003, never to C002.

#### Relationships

- **C002**: reads declared sources and hunts; writes config edits.
- **C003**: writes new tokens on `source add` / `rotate-token`.
- **C004, C006, C014**: writes registrations and edits.
- **C008, C009, C010, C011, C012, C013**: dispatches chase
  actions.
- **C015**: hands results off for rendering.

### C002 - Config Store

Reads and writes the single TOML configuration file. Holds source
declarations (`type`, `url`, non-secret auth fields, secret
reference), hunt declarations, and hunt scope selectors. Re-read
fresh on every CLI invocation. Holds no secrets.

#### Relationships

- **C001**: read by every command; written by config CRUD verbs.
- **C004, C006**: source the source and hunt declarations.

### C003 - Credential Store

Resolves a source secret via the OS keyring first, then the
`BIRDDOG_SOURCE_<NAME>_TOKEN` env-var fallback. Owns keyring
read/write paths for source tokens. The auth method (Basic,
Bearer, etc.) is the Source Adapter's concern; C003 only stores
and returns the secret string.

#### Relationships

- **C001**: writes new tokens on `source add` / `rotate-token`.
- **C005**: reads tokens at request time, scoped to one source.

### C004 - Source Registry

Holds the declared set of sources and resolves a source name to a
`(declaration, secret, adapter)` bundle on demand. Validates
source declarations on edit and on read.

#### Relationships

- **C002**: source declarations live here.
- **C003**: secrets are looked up here.
- **C005**: provides adapter instances bound to a source.
- **C006**: hunts reference sources by name, resolved here.

### C005 - Source Adapter

Source-type-specific contract for reading items from one external
source and reporting source availability. The Jira implementation
is the first realization; future source types implement the same
contract. Read-only.

#### Relationships

- **C003**: reads the bound source's secret per request.
- **C004**: instantiated against a registered source.
- **C008**: invoked by the Refresh Engine to pull items and
  report availability.

### C006 - Hunt Registry

Holds the declared hunts and their scope selectors. Each hunt is
a named collection of `(source, selector)` entries; selectors are
projects, JQL queries, or future source-type-specific scoping
shapes. Validates that referenced sources exist in C004.

#### Relationships

- **C002**: hunt declarations and selectors live here.
- **C004**: validates source references.
- **C008**: receives the hunt's scope at refresh time.

### C007 - Item Store

The State band's hub. Owns item identity (stable internal id,
distinct from source ids), the active set per hunt, manual items,
migration links, recorded expected trajectories, dependencies
between items, drop-off history, and rejected coverage
candidates. SQLite-backed.

#### Relationships

- **C008**: written on every refresh.
- **C009**: read for assessment, written with assessment readings
  and basis traces.
- **C010, C012, C013, C014**: hold their own per-item facts but
  reference items by C007's internal id.
- **C011**: read at synthesis time.
- **C015**: read at render time.

### C008 - Refresh Engine

For a target hunt, walks each `(source, selector)` entry, pulls
items via C005, updates C007 (new items, drop-offs, migration
links, source-availability state, set-level refresh marker), and
adopts source-side due dates as starting expected trajectories.
Surfaces coverage candidates — sources referenced from watched
data, and items in watched sources excluded from the active set.
Triggered on-demand by `birddog checkpoint` and by an explicit
`birddog refresh`.

#### Relationships

- **C005**: reads items and availability.
- **C006**: receives the hunt scope.
- **C007**: writes the resulting active set, drop-offs, migration
  links, and candidate sets.
- **C009**: invokes re-assessment on items whose underlying data
  has changed since their last assessment.

### C009 - Assessment Engine

Computes the per-item slip reading (trajectory comparison,
silence, combined slip status) and the per-item status reading
(basis, confidence, conflict, freshness) over C007's active set.
Carries "cannot assess," "cannot determine confidence," and
"sources disagree" states explicitly per Coverage-over-Precision.
Writes assessment readings and basis traces back to C007.

#### Relationships

- **C007**: reads the active set and underlying-data signals;
  writes assessment readings.
- **C008**: invoked on items flagged as changed since last
  assessment.
- **C010**: an active override is surfaced alongside the inferred
  reading; assessment never silently overwrites it.
- **C011**: produces the inputs synthesis consumes.

### C010 - Override Store

One persistent override mechanism, parameterized by the produced
judgment it covers: status (from C009), intervention
recommendation (from C011), and escalation timing recommendation
(from C011). Each override survives re-synthesis; new produced
output is surfaced alongside the active override for the
bird-dogger to adopt or dismiss. Cleared overrides remain
recoverable in history.

#### Relationships

- **C001**: written by `status`, `recommend`, and `escalate`
  override verbs.
- **C009, C011**: read at render time so the override stands as
  the displayed value with disagreement surfaced alongside.
- **C015**: receives override + produced output as a pair.

### C011 - Synthesis Engine

Reads C007 (state), C009 (assessment), C010 (overrides), C012
(prior triage), C013 (touch / escalation history), and C014
(escalation chain) to produce per-item urgency ranking,
intervention recommendation, and escalation timing
recommendation. Writes nothing back to its inputs; emits a
synthesis result consumed by C015.

#### Relationships

- **C007, C009, C010, C012, C013, C014**: read at synthesis time.
- **C015**: receives ranking and recommendations.

### C012 - Triage Memory

Records each item's prior triage decision (open vocabulary,
bird-dogger-supplied label) plus the snapshot of context at
decision time (rank, signals, recommendation). At the next
checkpoint, surfaces the prior decision alongside what has
changed since per the two-timestamp freshness shape.

#### Relationships

- **C001**: written by `triage` verbs.
- **C007**: references items by stable internal id.
- **C011**: read as a synthesis input.
- **C015**: read at render time.

### C013 - Touch Log

Append-only log of bird-dogger touches on items, including
escalation attempts: target (a chain member from C014), channel,
time, response state, and a label distinguishing "escalation
attempt" from "routine touch." Both labels share the same record
shape.

#### Relationships

- **C001**: written by `touch` and `escalate` verbs.
- **C007, C014**: references items and chain members by stable
  id.
- **C011**: read as a synthesis input — timing recommendation
  reads prior attempts and the response window.
- **C015**: read at render time for response-window display.

### C014 - Contact Registry

Per-item escalation contacts: the recorded owner and the chain
above. Tracks owner-confirmation freshness. Holds bird-dogger-
registered contacts and source-derived contacts side by side,
with disagreements surfaced rather than collapsed.

#### Relationships

- **C001**: written by `contact` verbs and by registration on
  `source add` (when source-derived owners enter).
- **C007**: references items by stable internal id.
- **C011**: read at synthesis time for escalation timing.
- **C015**: read at render time for chain display.

### C015 - Renderer

Produces tabular CLI output for list-level views and item-level
detail views. Carries the readability obligations from PRD001
(uncertainty / disagreement states stay readable, not buried) and
PRD002 (two timestamps surfaced distinctly per item). Writes only
to stdout/stderr; holds no state.

#### Relationships

- **C007, C009, C010, C011, C012, C013, C014**: read for the data
  to render.
- **C001**: invoked by the CLI to produce output.

## Requirement-Component Map

Every requirement is exposed via **C001 (CLI)** and rendered via
**C015 (Renderer)**. The map below names the additional
components that satisfy each requirement.

- **J001-O001-R001** — Proactive Slip Surfacing: C009, C008
- **J001-O001-R002** — Assessment Freshness: C009, C007
- **J001-O001-R003** — Expected Trajectory: C007, C008
- **J001-O001-R004** — Trajectory Comparison: C009
- **J001-O001-R005** — Silence As Signal: C009
- **J001-O001-R006** — Source Availability: C005, C008, C007
- **J001-O001-R007** — Source Registration: C004, C005, C003, C002
- **J001-O002-R001** — Coverage Set Visibility: C007, C004, C006
- **J001-O002-R002** — Manual Item Capture: C007
- **J001-O002-R003** — Coverage Gap Discovery: C008, C007
- **J001-O002-R004** — Drop-Off Surfacing: C008, C007
- **J001-O002-R005** — Item Migration Linking: C008, C007
- **J001-O002-R006** — Active Set Freshness: C008, C007
- **J001-O002-R007** — Source Registration: C004, C005, C003, C002
- **J001-O003-R001** — Status Basis: C009, C007
- **J001-O003-R002** — Status Freshness: C009, C007
- **J001-O003-R003** — Status Confidence: C009
- **J001-O003-R004** — Status Override: C010
- **J001-O003-R005** — Re-Assessment On Change: C008, C009
- **J001-O003-R006** — Conflict Surfacing: C009
- **J001-O004-R001** — Urgency Ranking: C011
- **J001-O004-R002** — List-Level Urgency Signals: (rendering of
  signals produced under O001 and O003 — no new component)
- **J001-O004-R003** — Intervention Recommendation: C011, C010
- **J001-O004-R004** — Triage State Carry-Forward: C012
- **J001-O004-R005** — Inter-Item Dependency Surfacing: C007
- **J001-O005-R001** — Escalation Target: C014, C007
- **J001-O005-R002** — Owner Currency: C014
- **J001-O005-R003** — Response Window Visibility: C013
- **J001-O005-R004** — Escalation Timing Recommendation: C011, C010
- **J001-O005-R005** — Escalation Record: C013
- **J001-O005-R006** — Escalation Contact Registration: C014
