# ADR004 - Selectors as First-Class Declarations

Selectors are promoted to first-class declarations alongside sources and
hunts. A selector is a named, reusable query against one source. Hunts
hold references to selectors by name as an unordered set, not inline
`(source, selector)` entries. Editing a selector propagates to every
hunt that references it on the next refresh.

## Context

[ADR001](ADR001-source-hunt-decoupling.md) framed the Connection band
as two axes — Source (auth) and Hunt (scope) — with hunt scope
expressed as inline `(source, selector)` entries. That framing made
selectors data attached to a hunt, not declarations in their own right.

In practice, the bird-dogger often wants the same query under several
hunts:

- A "Supply P0 issues" JQL appears under both an O001 slip-detection
  hunt (`supply-q3`) and an O005 leadership-summary hunt
  (`supply-leadership`).
- A "recently updated incidents" JQL feeds both an incident-triage
  hunt and a cross-team release-health hunt.
- A "items I own" filter is referenced from every hunt the bird-dogger
  actively works.

Under inline `(source, selector)` entries, the JQL has to be re-typed
into each hunt and kept in lockstep by hand. A typo in one hunt's copy
silently produces a different coverage set than its sibling. Engineering
README's C006 Hunt Registry holds the inline form; C002 Config Store
holds the duplicated text.

[PRD003](../../product/pdrs/PRD003-shared-source-registration.md) sets
the precedent for the analogous decision on the auth axis: one shared
capability (source registration) realized by one component, referenced
from many places, even when each consumer has its own outcome-layer
read. The scope axis has the same shape and was not yet captured.

Without a recorded decision, the architecture stays at the inline form
and the bird-dogger pays the duplication cost on every multi-hunt
scope.

## Options

- **A. Selectors as first-class declarations.** New `C017 Selector
  Registry` holds the declared set of selectors. Each selector is a
  named `(source, selector-type, value)` triple. `C006 Hunt Registry`
  holds an unordered set of selector-name references per hunt.
  `C002 Config Store` carries a selectors TOML table alongside the
  sources and hunts tables. `C008 Refresh Engine` resolves a hunt's
  selector names via `C017` at refresh time. Editing a selector
  propagates to every referencing hunt on the next refresh. Cost:
  one more top-level noun for the bird-dogger to learn; one more
  component in the Connection band; selector removal must check for
  referencing hunts (no cascade).
- **B. Keep selectors inline in hunt entries.** Status quo from
  ADR001. Cost: bird-dogger re-types the query into every hunt that
  needs it; sibling hunts drift on copy-edit; no shared edit path; no
  natural home for a `selector test` verb that exercises a query
  independent of any hunt.
- **C. Hunt-owned selectors with named export.** Each hunt declares
  its selectors inline, but selectors can be "exported" by name for
  other hunts to import. Cost: two parallel declaration shapes
  (inline + exported) that mean the same thing; the bird-dogger
  decides which to use per case; merge-edits across importing hunts
  become unclear (does the importer see the exporter's edit
  immediately? on next refresh? never?).

## Decision

Option A — selectors as first-class declarations.

The duplication cost in Option B is the same cost PRD003 ruled
against on the auth axis: shared capability, shared component, distinct
references. The scope axis behaves identically — the bird-dogger's
query library is a reusable asset, not a per-hunt copy. Option C
preserves the worst of both: two declaration shapes for the same data
and ambiguous propagation semantics.

ADR001's two-axis decoupling (auth ⟂ scope) stands. This ADR refines
the scope axis: scope is composed of named selectors, and the bird-
dogger declares each selector once. Hunts compose selectors by
reference, not by inclusion.

The promotion also gives `selector test` a home. Under the inline
form, testing a query meant either declaring a throwaway hunt or
running an ad-hoc API call. Under the first-class form, `bdog selector
test <name>` is a natural verb on a noun the bird-dogger already
maintains.

## Consequences

- New `C017 Selector Registry` in the Connection band. Holds declared
  selectors; resolves a selector name to a `(source, selector-type,
  value)` triple; validates that the referenced source exists in
  `C004`.
- `C006 Hunt Registry` simplifies: each hunt is a named set of
  selector names. The bugel walks the references and asks `C017`
  to resolve each; ordering between references is not
  bird-dogger-controlled.
- `C002 Config Store` carries a new TOML table for selectors. Source,
  selector, and hunt declarations live in three sibling tables; the
  bird-dogger can edit any of the three by text editor or by CLI.
- `C008 Refresh Engine` walks a hunt's selector references, resolves
  each via `C017`, and asks the bound `C005` Source Adapter to pull
  items per resolved triple. Source-side rate-limit pressure is
  unchanged from ADR001: one burst per refresh, bounded by the
  selectors in scope.
- CLI gets a new top-level `selector` noun with full CRUD plus
  `selector test`. `hunt selector` becomes a membership sub-resource
  (add / remove / list) that records the reference; the selector's
  data lives on the `selector` noun. Design doc owns the verb shape.
- Editing a selector propagates to every referencing hunt on the next
  refresh. The bird-dogger updates the query in one place; sibling
  hunts pick up the change without per-hunt edits. This is the
  primary motivating affordance.
- Selector removal requires no hunts reference it; cascade is
  rejected. The bird-dogger removes references first (`hunt selector
  remove`) and then removes the selector. Same posture as source
  removal under the no-cascade pattern this ADR composes into.
- One selector backs many hunts; one hunt mixes many selectors;
  selectors can cross sources within a hunt because each selector
  carries its own source binding. Cross-source hunts (anticipated in
  ADR001) work unchanged — the hunt's selectors can reference
  different sources.
- Identity at the item layer is unaffected: `C007` items are still
  source-scoped, not selector-scoped. The same Jira issue surfaced
  via two selectors in the same hunt is one item; selector
  attribution is a `C016` Coverage Memory concern when needed for
  per-source provenance.
- Composes with
  [PRD003](../../product/pdrs/PRD003-shared-source-registration.md):
  PRD003 shares the auth axis at the component layer; ADR004 shares
  the scope axis at the component layer. The two together let the
  bird-dogger declare one source plus one selector library and
  compose hunts freely.
- Selector versioning is explicitly deferred. There is no requirement
  to keep prior selector shapes for winding-down hunts; selectors
  remain mutable in place until a concrete requirement forces a
  versioning model. If that requirement arises, it will require a
  new decision — it is not on the roadmap.
