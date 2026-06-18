# 2026-06-16 — CLI Design Doc and Engineering README Review

Read-only review session. Studied the high-level product docs
(product README, 4 PDRs, 5 outcome READMEs — not the R-docs), the
CLI design doc at `docs/engineering/design-docs/cli.md`, the
engineering root at `docs/engineering/README.md`, and the 5 ADRs.
Identified anti-patterns and inconsistencies; produced options for
resolution; recommended a path forward. No files edited.

## Purpose

Pressure-test the relationship between the architecture root and
the CLI design doc. The CLI design doc claims to "supersede the
placeholder verb list" in the engineering README, but the README
isn't a placeholder anymore — it carries detailed verb and flag
references in its component descriptions. The session asked
whether the two documents are consistent and whether each holds
content at the right altitude.

## Outcome

Eleven findings grouped into themes. A two-phase recommendation
(promote weight-bearing decisions to ADRs, then strip CLI verb
detail out of the engineering README's component descriptions),
preceded by discrete factual fixes. No edits made in the
analysis session itself — the next agent picks up from this
digest and executes.

## Update — 2026-06-16 follow-up

User reviewed and resolved the four open questions before
Phase 1 execution. Decisions captured below; tasks below have
been edited in place to reflect them.

- **Task 1.1 — Jira source type:** two types, `jira-cloud`
  (Atlassian-hosted) and `jira-dc` (self-hosted Data Center).
  `local-json` remains the third first-class type. README,
  ADR005 Context, and C005 description name all three.
- **Task 1.2 — C015 dual role:** option (c). List-level signal
  selection moves to C011's synthesis output shape; C015 stays
  a pure renderer. C011 emits a per-item list-level signal pack
  alongside its urgency ranking and recommendations. The
  requirement-component map points R002 at C011 (with a
  breadcrumb that the signals themselves originate in O001/O003,
  as before).
- **ADR008 framing:** single ADR covering both postures
  (deactivate vs. hard-delete) with the rule for which record
  types get which. Reads cleaner than splitting into two ADRs.
- **`--clear-<field>` shape graduates to an ADR.** Added as a
  sixth Phase 2 task — ADR011 (Two clearing shapes). It's a CLI
  ergonomic but it constrains every `set` verb that accepts
  clearable values and forecloses some shapes (e.g.,
  `--clear-token`-style flags) for the secret model, so it
  belongs above the design-doc layer.

Phase 2 now plans **six** ADRs (ADR006–ADR011), not five.

### Phase 1 — executed

All six Phase 1 tasks were applied in the follow-up session.
Concrete edits:

- **Task 1.1.** README intro line, README Constraints (source-type
  list rewritten), README C005 description (names all three
  types), ADR005 Context (acknowledges jira-cloud / jira-dc as
  one adapter family that does not exercise the
  source-type-agnostic claim).
- **Task 1.2.** README C011 (now emits per-item list-level signal
  pack), README C015 (rendering-only), README requirement-
  component map line for J001-O004-R002 (points to C011), cli.md
  Renderer surfaces opening (attributes both C015 rendering
  obligations and C011 signal-selection obligation), cli.md
  design-decision "Renderer surfaces are part of the design" entry
  (same reattribution).
- **Task 1.3.** README C002 (id field shape, cross-references key
  on id, CLI is the canonical id assigner), cli.md new
  `### Hand-edit reconciliation` subsection inside Renames (rules
  for adding new entities by hand, editing existing entities by
  hand, and the cross-reference id-form requirement).
- **Task 1.4.** README opening paragraph (one-clause intro of
  bugel as the CLI's chase-metaphor name for the checkpoint
  moment, with link to the CLI design doc).
- **Task 1.5.** README "One freshness contract" principle
  (per-item realizes PRD002, set-level marker called out as the
  cousin shape PRD002 acknowledges but does not commit to).
- **Task 1.6.** README Constraints (dropped duplicate
  "read-only access to external sources"; the Principles entry
  citing ADR003 is the single home).

Phase 2 (ADR006–ADR011) and Phase 3 (verb-detail strip from
README) remain pending. Each Phase 2 ADR is its own
draft → judge → apply session per the intent workflow.

## Inputs Loaded

For continuity if a future session needs to rebuild context:

- `docs/product/README.md`.
- All 4 PDRs under `docs/product/pdrs/`.
- All 5 outcome READMEs under `docs/product/outcomes/`.
- `docs/engineering/README.md` (architecture root).
- `docs/engineering/design-docs/cli.md` (CLI design doc).
- All 5 ADRs under `docs/engineering/adrs/`.
- Intent skill `SKILL.md` (for framework conformance).
- Prior session digests (`2026-06-10-engineering-root-draft.md`,
  `2026-06-12-engineering-root-judge.md`) — for continuity on
  what's already been judged.

R-docs were intentionally not loaded — the user scoped the review
to high-level docs only.

## Findings

### Theme 1 — CLI surface bleeding into the architecture README

The CLI design doc opens with "Supersedes the placeholder verb
list in the [engineering root]" (`cli.md:6-7`), but the engineering
README still carries verb-by-verb references in component
descriptions:

- `README.md:145-159` (C001) — enumerates CRUD nouns, action verbs,
  bugel semantics.
- `README.md:359-360` (C010) — `bdog override` parameterized by
  `<field>` enum.
- `README.md:400-401` (C012) — `note` verbs (add/remove/info/list/set).
- `README.md:424-425` (C013) — `touch` verbs and the `type` field.
- `README.md:457-460` (C014) — `contact`, `chain`, `chain member`,
  `item set --owner`, `item set --chain`, `item confirm-owner`.
- `README.md:256-259` (C006) — `bdog hunt bugel`, `hunt activate`,
  `hunt deactivate`, `hunt selector` membership verbs.

When the CLI changes, both docs must be updated. The "supersedes"
claim implies one-way authority that the README doesn't honor.

### Theme 2 — Native id scheme is a CLI-doc decision but system-wide

`cli.md:79-90` commits to `bdogitem-<n>`, `bdognote-<n>`,
`bdogtouch-<n>`. README C007 only says "stable internal id
(distinct from source ids)" without committing to the scheme.

The scheme shapes C007, C012, C013, C016 keying, log messages,
error messages, and any future State-band component. It's an
architecture-level identity decision currently owned by the CLI
design doc. Engineering-to-CLI is the wrong direction for
identity.

### Theme 3 — CLI doc's "Design decisions" section is 17 embedded micro-ADRs

`cli.md:1034-1188` reads like an ADR catalog without the ADR
shape (no Context / Options / Consequences). Some entries are
pure presentation (noun-first, `add` vs `set`, two clearing
shapes). Others are weight-bearing engineering decisions that
should be ADRs:

- Native `bdog<kind>-<n>` id scheme with collision-fail
  (`cli.md:1057-1067`).
- No-cascade removal (`cli.md:1101-1106`).
- `override remove` deactivates; `touch`/`note remove` hard-delete
  (`cli.md:1117-1122`).
- Side-effecting actions on their own verbs, not `set`
  (`cli.md:1146-1156`).
- Tokens never appear as CLI flags (`cli.md:1140-1144`).

### Theme 4 — Enum duplication

- **Selector types** (`project|jql|filter`): `README.md:518` and
  `cli.md:269,275,281`.
- **Override fields** (`status|intervention|escalation-timing`):
  `README.md:359-360` and `cli.md:560,578-582`.
- **Touch type** (`escalation|checkin`): `README.md:415-417` and
  `cli.md:605,619,626`.

Adding a value means updating both files.

### Theme 5 — Source-type enum mismatch (factual)

- CLI doc: `jira-cloud|jira-dc|local-json` (`cli.md:230`).
- README: "Jira" + `local-json` (`README.md:106-107`) — two types,
  not three.
- ADR005: "Jira is the only concrete adapter declared so far"
  (singular).

Either jira-cloud and jira-dc are two adapters (README + ADR005
need updates) or they're one adapter with different URL
conventions (CLI doc needs the split removed). The architecture
hasn't declared which.

### Theme 6 — C015 has two roles described as one (factual)

`README.md:468-484` says C015 "writes only to stdout/stderr;
holds no state" AND "owns list-level signal selection per
J001-O004-R002 (which urgency-driving facts appear on each row,
at what density)."

Signal selection is a composition decision, not rendering.
Either C015 has two responsibilities (renderer + list composer),
or signal selection belongs on C011's output shape. Worth a
deliberate call.

### Theme 7 — TOML hand-edit + stable-id reconciliation gap

`cli.md:117-128` explains renames are safe because storage keys
on ids, and `info`/`list` surface the id "so a bird-dogger
editing the TOML by hand can match either form."

But:
- `README.md:175-184` (C002) describes the TOML schema without
  mentioning ids.
- Neither doc specifies the TOML id field format.
- Neither doc says who assigns an id when a bird-dogger adds an
  entity by hand-editing the TOML.

"Both paths mutable" (`README.md:113-115`) + stable cross-refs
keyed on ids = an unspecified flow at the hand-edit boundary.

### Theme 8 — "Bugel" terminology gap

README uses `bdog hunt bugel` throughout without ever explaining
it. CLI doc explains once (`cli.md:1174-1177`) that bugel is a
colloquialism for the product-level "checkpoint." A reader going
README → CLI doc has to backtrack to learn the term.

### Theme 9 — README C002 and C014 hold detail that belongs lower

- **C002** (`README.md:175-184`) names TOML field names — schema
  fragment in the architecture root.
- **C014** (`README.md:447-451`) explains *why* it lives in the
  State band — ADR rationale embedded in architecture.

### Theme 10 — README freshness principle expands PRD002

PRD002 commits to per-item two-timestamp; calls the set-level
marker a "cousin shape." README principle (`README.md:50-52`)
folds the set-level marker into "Realizes PRD002" — slight
overclaim. Easy wording fix.

### Theme 11 — Smaller items worth naming

- **Principles vs Constraints overlap.** "Read-only against
  sources" appears in both (`README.md:81-85` and `:104-105`).
- **No `docs/engineering/components/` directory** — all 16
  components inline in the README. Allowed by the intent
  framework's simple template, but the README is 578 lines.
- **No breadcrumb from C015 to renderer surfaces.** A reader at
  C015 doesn't know the per-verb output spec lives in
  `cli.md:898-1032`.
- **`item set --hunt` is manual-only**, but `item set --title`
  works on source-derived items too — asymmetry not called out
  (does `--title` overwrite the rendered title? sync? local
  alias?).

## Options Considered

Composable, not mutually exclusive.

### Option A — Strip CLI verb detail out of the README

Component descriptions name capabilities ("manual item capture,"
"owner confirmation") instead of verbs. The CLI design doc is
the only place verbs are spelled out.

- **Cost:** README becomes more abstract; harder to trace
  component → CLI without a doc jump.
- **Benefit:** one verb change updates one file. README stays at
  architectural altitude.

### Option B — Promote weight-bearing decisions to ADRs

Pull the system-wide decisions out of the CLI doc's "Design
decisions" section into proper ADRs (Context / Options /
Decision / Consequences). Candidates:

- **ADR006 — Native id scheme.** `bdog<kind>-<n>`; source ids as
  aliases; collision-fail.
- **ADR007 — No-cascade removal.** Architectural posture across
  declarative entities and parents-of-references.
- **ADR008 — History posture per record type.** Overrides
  deactivate (PRD004); touches/notes hard-delete. Single ADR
  covering both postures.
- **ADR009 — Side-effecting actions distinct from `set`.**
  State-machine and timestamp-advancing actions don't live on
  `set`.
- **ADR010 — Tokens never via CLI flag.** Interactive-prompt or
  env-var fallback only.
- **ADR011 — Two clearing shapes.** `none` sentinel for typed
  reference fields; `--clear-<field>` for free-text. One rule
  per field type, applied uniformly across every `set` verb
  with clearable fields.

- **Cost:** 6 ADRs to write; some are partly archaeology of
  decisions never rigorously enumerated up front. The Alternatives
  sections will be partly synthesized rather than recovered.
- **Benefit:** decisions land at the right altitude with proper
  shape. README and CLI doc both reference; neither owns in two
  places.

### Option C — Fix only the discrete factual inconsistencies

Leave the architecture-vs-CLI overlap; fix the specific things
that are wrong or undefined:

- Jira source-type question (one or three types?).
- C015 dual role (split, move, or rename).
- TOML id field format and hand-edit reconciliation.
- "Bugel" introduction in the README.
- "Realizes PRD002" wording on the set-level marker.

- **Cost:** duplication and inverted-altitude problems remain;
  re-emerge on next verb/component change.
- **Benefit:** smallest edit surface; lowest narrative risk.

### Option D — Restructure into per-component docs

Move each component to `docs/engineering/components/C<NNN>-<name>.md`.
Architecture README becomes a thin shell. C001 and C015 get their
own docs that own CLI and renderer surfaces; the design-doc file
retires.

- **Cost:** large reorganization; redirects to update; the
  design-doc artifact disappears.
- **Benefit:** structurally aligned with the intent framework's
  expected layout.

## Recommendation

**C + B + A, in that order. Defer D.**

1. **Do C first** — discrete fixes are cheap and resolve actual
   factual gaps.
2. **Do B next** — promote the 5 weight-bearing decisions into
   ADRs. The native id scheme especially is too foundational to
   live in a CLI design doc.
3. **Do A last** — strip verb/flag references from the README's
   component descriptions. Replace with capability language. Add
   one breadcrumb at the top of C001 ("see CLI design doc for the
   full verb surface") and one at the top of C015 ("see CLI design
   doc for renderer surfaces").

**Why defer D.** It's the most structurally pure answer but it
spends a lot of reorganization cost for benefits that materialize
only when per-component detail accumulates. The codebase doesn't
have per-component ADRs or CRs yet (per the 2026-06-12 digest,
those are planned as part of a future components pass). B+A
captures most of the win at a fraction of the cost and leaves D
as a clean future move.

**Tradeoffs the next agent should hold.**

- **B's archaeology problem.** Several of these decisions were
  made without rigorous alternatives enumeration. Be honest about
  this: the Alternatives sections will be partly "the obvious
  alternatives the design implicitly rejected," not "the options
  we evaluated at the time." This is acceptable per the intent
  framework's candor guidance — flag synthesized alternatives as
  such.
- **A makes the README less browseable.** Today a reader at C014
  sees "written by `contact` and `chain` declaration verbs, by
  `chain member` membership verbs, by `item set --owner` /
  `item confirm-owner`, and by `item set --chain`" and knows
  immediately how the component is driven. After A, they see
  capability language and have to follow a link. That's the cost
  of altitude separation.
- **B's CLI-presentation boundary.** Picking which design-decisions
  graduate to ADRs is judgment. The original analysis pushed
  back on `--clear-<field>` as borderline; the user resolved it
  upward — promote. Future close calls (e.g., the
  `chain member set --position` shift behavior) should lean
  toward promotion when the decision constrains components other
  than C001/C015 or forecloses adjacent shapes.

## Next Steps (executable task list)

The next agent should be able to work these to completion. Each
task has enough context to execute without rebuilding the whole
analysis.

### Phase 1 — Discrete factual fixes (Option C)

**Task 1.1 — Resolve Jira source-type question. [Decided: two types]**
- User decision: two source types, `jira-cloud` and `jira-dc`,
  with `local-json` remaining as the third first-class type.
- README (`README.md:106-107`) — rewrite the source-type sentence
  to enumerate three types; explain that Cloud and DC are
  distinct types because they differ in auth, endpoints, and
  rate-limit model despite sharing query semantics.
- ADR005 Context (`ADR005:15-16`) — update "Jira is the only
  concrete adapter declared so far" to acknowledge the
  jira-cloud / jira-dc split. Leave the Decision and Consequences
  intact (they're about local-json's first-class status).
- C005 description (`README.md:223-232`) — name all three types;
  note that the Jira split is by source type, not by adapter
  contract. The C005 contract is one; three types realize it.

**Task 1.2 — Resolve C015 dual role. [Decided: option (c)]**
- `README.md:468-484` claimed C015 was a stateless renderer AND
  owned list-level signal selection. The latter is a composition
  decision, not a rendering one.
- Move list-level signal selection to C011's synthesis output
  shape:
  - Update C011's responsibility (`README.md:365-376`) to add an
    emitted per-item list-level signal pack — which urgency-
    driving facts to surface alongside the ranking and
    recommendations — per J001-O004-R002.
  - Update C015's responsibility (`README.md:468-477`) to
    describe it as pure rendering: lays out what synthesis hands
    it; carries PRD001/PRD002 readability obligations at the
    rendering layer; no composition decisions.
  - Update the requirement-component map (`README.md:564-566`)
    to point R002 at C011, with the breadcrumb preserved that
    the signals themselves are produced under O001 and O003.

**Task 1.3 — Specify TOML id field and hand-edit reconciliation.**
- Add a paragraph to `README.md` C002 (`README.md:175-184`)
  describing the id field shape in the TOML schema.
- Add a paragraph to `cli.md` (probably in the "Renames" section
  around line 108) describing what happens when a bird-dogger
  adds a new entity by hand-editing the TOML: when is the id
  assigned, by which CLI invocation, and what happens to
  cross-references in the interim.
- This may need an ADR if the reconciliation logic is non-trivial.

**Task 1.4 — Introduce "bugel" in the README.**
- Add one sentence to the README — either in the intro near
  `README.md:8` or in C001 (`README.md:145-159`) — naming bugel
  as the CLI surface for the product's "checkpoint" concept. One
  line is enough; the full explanation can stay in the CLI doc.

**Task 1.5 — Tighten "Realizes PRD002" wording.**
- `README.md:50-52` claims the freshness principle realizes
  PRD002 in full, but PRD002 only commits to per-item; the
  set-level marker is a cousin shape acknowledged in PRD002, not
  committed to.
- Reword to: "Realizes [PRD002] for the per-item shape; extends
  it with a separate set-level marker for active-set freshness
  (cousin shape PRD002 acknowledges)."

**Task 1.6 — Fix Principles/Constraints overlap.**
- "Read-only against sources" appears at `README.md:81-85`
  (Principles) and `:104-105` (Constraints). Pick one home; the
  Principles section is the natural fit since it cites ADR003.
  Remove from Constraints.

### Phase 2 — Promote weight-bearing decisions to ADRs (Option B)

Each is a standalone modification session per the intent workflow
(draft → judge → apply). Don't batch — each ADR deserves its own
Context / Options / Decision / Consequences pass.

**Task 2.1 — ADR006 — Native id scheme.**
- Source material: `cli.md:79-90` and `cli.md:1057-1067`.
- Context: identity has to work across manual and source-derived
  items, across multiple sources, in scripts and logs.
- Options to enumerate (synthesize honestly): native scheme with
  kind prefix (chosen); UUIDs; source-id-only with collision
  handling; per-kind random tokens.
- Consequence: the scheme is referenced from C007, C012, C013,
  C016, and every CLI verb that takes an `<item>` arg.
- After landing: update README C007 to reference ADR006 instead
  of the bare "stable internal id" claim.

**Task 2.2 — ADR007 — No-cascade removal.**
- Source material: `cli.md:130-162` and `cli.md:1101-1106`.
- Context: removing a declarative entity could (a) silently
  cascade through consumers, (b) fail fast and require the
  bird-dogger to tear down references first, or (c) prompt
  interactively.
- Note the partial-cascade reality: source-derived items
  disappear silently when a hunt is removed, but manual items
  must be re-assigned first (`cli.md:157`, `cli.md:429-431`). The
  ADR should name and defend this asymmetry.

**Task 2.3 — ADR008 — History posture per record type. [Decided: single ADR]**
- Source material: `cli.md:130-145` and `cli.md:1117-1122`.
- Context: PRD004 demands override history survives `remove`;
  touches and notes are corrective records with no history
  obligation.
- Framing: single ADR. Names the two postures (deactivate vs.
  hard-delete), records the rule for which record types get
  which, and defends the asymmetry. Splitting into two ADRs was
  considered and rejected — the postures only make sense when
  read against each other.

**Task 2.4 — ADR009 — Side-effecting actions distinct from `set`.**
- Source material: `cli.md:1146-1156`.
- Context: the noun-first sub-verb pattern (`add|remove|info|list|set`)
  doesn't fit timestamp-advancing or state-machine-transitioning
  actions. The ADR defends the action-verb carve-out architecturally,
  not just at the CLI surface.

**Task 2.5 — ADR010 — Tokens never via CLI flag.**
- Source material: `cli.md:1140-1144`, `cli.md:213-219`,
  `README.md:96-100`.
- Context: tokens passed as flags appear in shell history and
  process lists. The ADR commits to interactive prompt + env-var
  fallback, and excludes the flag path entirely.
- This constrains C003 (Credential Store) and C005 (Source
  Adapter), not just the CLI verb shape.

**Task 2.6 — ADR011 — Two clearing shapes (`none` vs `--clear-<field>`).**
- Source material: `cli.md:57-68`, `cli.md:1108-1115`,
  `cli.md:319-336`.
- Context: clearing a field could (a) accept a sentinel value
  (`<flag> none`), (b) use a parallel clear flag
  (`--clear-<field>`), (c) accept an empty string, or (d) require
  a separate verb. Each has a different collision surface against
  legitimate values.
- Decision the ADR records: typed reference fields use the
  `none` sentinel (target is an entity name; `none` is reserved);
  free-text fields use `--clear-<field>` (a real value of `"none"`
  is legal). One rule per field type, applied uniformly across
  every `set` verb that accepts a clearable.
- Why this is ADR territory rather than CLI-design-doc territory:
  it constrains every component whose `set` verb takes a
  clearable field, and it forecloses adjacent shapes the secret
  model could otherwise have wanted (e.g.,
  `--clear-token` on `source set` is foreclosed by ADR010 +
  this decision composing — tokens never on flags, period).
  Promote the rule above the CLI surface so future verbs inherit
  it without re-deciding.

### Phase 3 — Strip CLI verb detail from the README (Option A)

Single sweep across the README. Don't split into per-component
tasks — the rewrite is uniform.

**Task 3.1 — Inventory CLI verb references in the README.**

Locations to revisit:
- `README.md:145-159` (C001) — whole paragraph.
- `README.md:256-259` (C006) — `bdog hunt bugel`, `hunt activate`,
  `hunt deactivate`, `hunt selector`.
- `README.md:359-360` (C010) — `override` verbs with `<field>`.
- `README.md:400-401` (C012) — `note` verbs.
- `README.md:424-425` (C013) — `touch` verbs.
- `README.md:457-460` (C014) — `contact`, `chain`, `chain member`,
  `item set --owner`, `item set --chain`, `item confirm-owner`.
- `README.md:518-522` (C017) — `selector` verbs and `selector test`.

**Task 3.2 — Replace verb names with capability language.**

Examples of the rewrite shape:
- Before: "written by `contact` and `chain` declaration verbs, by
  `chain member` membership verbs, by `item set --owner` /
  `item confirm-owner`, and by `item set --chain`."
- After: "written by contact and chain declarations, by chain
  membership edits, by item-owner assignment and confirmation,
  and by item-chain assignment."

The capability nouns should match the language the design doc
already uses in its prose (not the flag names).

**Task 3.3 — Add two breadcrumbs.**
- C001 (top of `README.md:139-159`): one sentence pointing to
  the CLI design doc for the full verb surface.
- C015 (top of `README.md:468-484`): one sentence pointing to the
  CLI design doc's "Renderer surfaces" section for per-verb
  output shapes.

**Task 3.4 — Drop the duplicated enums from the README.**

- `README.md:518` (selector types) — let the CLI doc own.
- `README.md:359-360` (override fields) — let the CLI doc own.
- `README.md:415-417` (touch type field) — leave the existence
  of the type field, drop the enum values.

## Open Questions Carried Forward

Resolved in the 2026-06-16 follow-up (see Update section above):
Task 1.1 source-type split (two types), Task 1.2 C015 framing
(option c), ADR008 framing (single ADR), `--clear-<field>`
graduation (yes — ADR011).

Still carried forward:

- **Hand-edit reconciliation may need its own ADR.** Task 1.3
  documents the rule (id field shape; lazy id assignment on
  next CLI touch when a bird-dogger hand-adds an entity to the
  TOML; cross-references accept either name or id). The lazy
  assignment + write-back behavior is non-trivial and may warrant
  ADR012. Leave as a Phase 1 doc edit; flag for Phase 2 review.
- **`item set --title` on source-derived items.** Behavior
  undocumented. Worth a separate small clarification — local
  alias? overwrites at render? — but this is a CLI design doc
  fix, not a separate ADR.
- **Selector-versioning open question from ADR004** (`ADR004:137-140`)
  is unrelated to this session but flagged here for completeness
  if a future agent works through ADR004's followups.

## Files Touched

The original analysis session was read-only. The 2026-06-16
follow-up applied all six Phase 1 edits to:

- `docs/engineering/README.md` (intro paragraph, "One freshness
  contract" principle, Constraints source-type rewrite + read-only
  duplicate dropped, C002 id-field paragraph, C005 three-type
  description, C011 signal-pack emission, C015 rendering-only
  framing, requirement-component map R002 retargeting).
- `docs/engineering/design-docs/cli.md` (Renderer surfaces
  opening attribution; new `### Hand-edit reconciliation`
  subsection inside Renames; design-decisions "Renderer surfaces"
  entry attribution update).
- `docs/engineering/adrs/ADR005-local-json-source-type.md`
  (Context section acknowledges jira-cloud / jira-dc split).

The next agent picks up at Phase 2.
