# C015 — Renderer

Produces tabular CLI output for list-level views and item-level detail
views. Per-verb output shapes are specified in this document. Carries
the readability obligations from
[PRD001](../../product/pdrs/PRD001-coverage-over-precision.md)
(uncertainty and disagreement states stay readable, not buried) and
[PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md)
(two timestamps surfaced distinctly per item) at the rendering layer —
column layout, density given table width, overflow behavior, marker
glyphs. Renders the per-item signal pack C011 emits; does not pick
which signals matter (that is synthesis). Writes only to stdout/stderr;
holds no state.

## Readability obligations

Three PRDs bind C015 concretely:

- **PRD001 (Coverage over precision).** Uncertainty states
  (`cannot-assess`, `not-yet-assessed`, `cannot-determine`,
  `sources-disagree`, `cannot-recommend`) render as first-class column
  values — never blank, never collapsed to a dash. Disagreement between
  registered and source-derived values is surfaced inline rather than
  hidden.

- **PRD002 (Per-item freshness presentation).** Two timestamps are
  carried and displayed distinctly per item: `last data change`
  (updated by C008 on refresh) and `last assessed` (updated by C009 on
  assessment). These are never merged into a single "last updated."

- **PRD004 (Override surfacing).** When an active override exists for
  any produced judgment (status, intervention recommendation, escalation
  timing recommendation), the override value renders as the displayed
  value with a `*` marker; the inferred synthesis output renders in
  parentheses when it differs. Deactivated overrides surface via
  `--show-history`.

## Renderer surfaces

The verb output shapes below are part of the design, not
implementation detail. Per the CLI design
doc's [Renderer surfaces](../design-docs/cli.md) section, C011 carries
the list-level signal-selection obligation (J001-O004-R002); C015
renders whatever signal pack C011 emits.

### `bdog hunt bugel`

The primary triage surface. Composed sections, in order:

1. **Header.** Hunt name(s), the set-level last-refresh timestamp
   (PRD002 cousin shape; per-hunt when multiple hunts run), and
   a one-line source-availability summary (`2/3 sources available`).

2. **Source availability detail** (only when any source is
   unavailable). Lists the unavailable sources, the reason
   (network-error, file-missing, unparseable, auth-failed), and the
   affected hunts. Items from unavailable sources surface in the main
   table marked `source-unavailable` per J001-O001-R006 — never blank,
   never assumed-healthy.

3. **Active items table**, ordered by urgency rank. Columns:
   - `id` — native id (`bdogitem-N`); source id shown as `(SUPPLY-1234)`
     suffix when present.
   - `title` — truncated.
   - `slip` — slip status. Accepts uncertainty states `cannot-assess`
     and `not-yet-assessed` as first-class values per PRD001.
   - `status` — status reading. When an override is active, the
     override value renders as the displayed status with a `*` marker;
     the inferred reading renders in parentheses when it differs (PRD004
     disagreement surface). Uncertainty states (`cannot-determine`,
     `sources-disagree`) are first-class values per PRD001.
   - `intervention` — recommendation. Same override-marker and
     uncertainty rules apply (`cannot-recommend`).
   - `data` — last-underlying-data-change timestamp, relative
     (`2h ago`, `3d ago`); literal `unknown` when the source does not
     expose one (PRD001 + PRD002).
   - `assessed` — last-assessed timestamp, relative; literal `not-yet`
     when never assessed.
   - `owner` — current owner's display name; `!` suffix when the
     source-derived owner differs from the registered owner (PRD001
     disagreement on contacts); `?` suffix when owner-confirmation is
     older than a configurable freshness window (J001-O005-R002 Owner
     Currency).

4. **Notes recap** (collapsed by default; expanded with `--show-notes`
   once the flag is added — out of scope for now). At minimum, the
   table marks rows with a `note` indicator when the item has any
   active notes; the bird-dogger drills into `bdog item info` to read
   them.

### `bdog item info`

Stacked sections:

- **Identity.** Native id, source id (if any), origin (manual or
  source-derived), title, owning hunt (for manual items), current
  active overrides (one line per field).
- **Freshness.** Two timestamps labeled distinctly: `last data change`
  and `last assessed`. Both shown with absolute and relative formats.
  PRD002 obligation: these are never merged into a single "last
  updated."
- **Slip & status.** Current slip reading, status reading, basis
  summary. With `--show-basis`, expands to per-source inputs and any
  disagreement detail; uncertainty states render with their explanation
  ("not yet assessed because trajectory not recorded").
- **Overrides.** For each field with any history: current state (active
  or inactive), value, when set, reason, and the current inferred
  reading from synthesis when it differs (PRD004 surface). With
  `--show-history`, expands to full override history including
  deactivated entries.
- **Recommendations.** Intervention and escalation-timing
  recommendations, with `cannot-recommend` as a first-class value and
  active overrides surfaced alongside.
- **Owner & chain.** Current owner, last-confirmed-at; source-derived
  owner when it differs (PRD001 disagreement surface). Chain name and
  ordered members.
- **Dependencies.** Inter-item dependencies (`depends-on`) and reverse
  dependencies (`depended-on-by`).
- **Recent notes.** Last N notes (default: 5) newest-first, each with
  text, write timestamp, and one-line context snapshot.
  `--show-history` includes all notes.
- **Recent touches.** Last N touches (default: 5) newest-first, each
  with type, channel, target, time, message, response, derived response
  window. `--show-history` includes all touches.

### `bdog item list`

Same column set as the bugel's active-items table; no header section.
Intended for ad-hoc scanning when the bird-dogger does not need the
full bugel composition.

### `bdog hunt info` / `bdog hunt list`

- `bdog hunt info <hunt>` shows hunt name, active flag, set-level
  last-refresh timestamp, the associated selectors (with each
  selector's source and type for context), per-source availability
  state from the most recent refresh, and a count summary (active
  items).
- `bdog hunt list` shows one row per hunt: name, active flag,
  last-refresh, active-item count.

### Other list/info verbs

`source`, `selector`, `contact`, `chain`, `override`, `touch`, `note`
`list` and `info` follow conventional CRUD rendering — labeled fields
for `info`, columnar table for `list` with identity column first.
Uncertainty rendering does not apply to declarative entities (`source`,
`selector`, `contact`, `chain`) because they carry no produced
readings; it does apply to `override`, `touch`, and `note` only insofar
as they reference items whose readings could be uncertain.

## Relationships

- **C007, C009, C010, C011, C012, C013, C014, C016**: read for the
  data to render.
- **C001**: invoked by the CLI to produce output.

## Success criteria

- Uncertainty and disagreement states render as first-class column
  values; no blank cells for `cannot-assess`, `sources-disagree`,
  `cannot-recommend`.
- Two freshness timestamps (`last data change` and `last assessed`)
  appear as distinct labeled columns — never merged.
- Active overrides render with `*` marker; differing inferred value
  in parentheses.
- Bugel composition is: header → source availability (when needed) →
  active items table → notes recap.
- `hunt info` count summary shows active items only.
- Tabular output respects terminal width; no wrapping that loses
  column alignment.
