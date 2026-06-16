# 2026-06-12 — Engineering Root Judge

Judge session on `docs/engineering/README.md` per the intent
skill's Architecture rubric. The session both judged and applied
edits in-place — 11 of 12 flagged issues landed as edits; one
(ADR backfill) deferred to its own modification session.

## Purpose

Pressure-test the architecture root from the previous session
(2026-06-10) against the intent Architecture rubric (structural;
requirement-driven; aligned with ancestors; internally coherent)
plus the global Plain Language and Candor guidance, then apply
the resulting edits in the same session.

## Outcome

The architecture document passes the rubric. Twelve wobbles
identified; eleven resolved with edits inside `docs/engineering/
README.md`; one (ADR backfill) deferred.

The component count went from 15 to 16 (new C016 Coverage
Memory). The per-item / per-hunt axis is now the explicit seam
through the State band.

## Inputs Loaded

For continuity if a future session needs to rebuild context:

- `docs/engineering/README.md` (the document under judgment).
- `docs/product/README.md` and all 5 outcome READMEs.
- All 4 PDRs under `docs/product/pdrs/`.
- Three R-docs pulled in full for cross-check: `J001-O004-R002`
  (list-level urgency signals), `J001-O002-R003` (coverage gap
  discovery), and both `R007`s (source registration — to verify
  the identical-wording / one-bundle claim).
- Intent skill: `SKILL.md`, the architecture template, and the
  three `references/` files (context-tracing, structure,
  workflows).
- Previous session digest (`2026-06-10-engineering-root-
  draft.md`) — author's flagged pressure points used as judging
  probes.
- `resources/bird-dogging/bird-dogging-overview.md`.

## Rubric Verdict

| Rubric line | Verdict |
| --- | --- |
| Structural, not feature-focused | Pass. 4 bands + 16 components + reciprocal relationships. |
| Carries every requirement claimed | Pass. All 31 requirements mapped. |
| Aligned with ancestors | Pass. 4 PDRs realized as principles; no contradictions with product / outcomes. |
| Internal coherence | Pass after edits. 11 wobbles closed; band placement, principle wording, and relationship reciprocity all tightened. |
| Plain language | Pass. Short sentences, common words. |
| Candor | Pass after edits. Deferred decisions now explicitly flagged as ADR territory inside responsibility paragraphs. |

## Issues and Resolutions

| # | Issue | Resolution |
| --- | --- | --- |
| 1 | C007 (Item Store) breadth — 8 concerns under one component, no defense | **Split** along the per-item / per-hunt axis. C007 keeps per-item facts (identity, origin tag, migration links, trajectories, dependencies, plus the assessment readings C009 writes back). New **C016 Coverage Memory** owns per-hunt coverage state (active set, drop-offs, candidate items/sources, source-availability, set-level refresh marker). Updated bands intro, C007/C008/C009/C011/C015 responsibilities and relationships, and 7 map entries. |
| 2 | Per-item "last underlying data change" timestamp had no named owner | Named on **C007** (stored alongside assessment readings); **C008** writes it on every refresh. Principle line ("One freshness contract") unchanged — the wording already covered the shape; only the storage owner was unnamed. |
| 3 | PRD002's multi-source picking rule unacknowledged | Flagged inline in **C008's responsibility** as ADR territory ("max-of, per-source array, composite"). C008 holds the write site once the rule lands. |
| 4 | C001's chase-verb list read as exhaustive but wasn't | Extended the named-verb list with `refresh`. Added a sentence naming the deferred chase-verb capabilities with R-doc citations: expected-trajectory recording (R003/O001), manual item capture (R002/O002), coverage-candidate accept/reject (R003/O002), dropped-item carry-forward (R004/O002), inter-item dependency recording (R005/O004). Exact verb syntax flagged as ADR territory. Added C007 + C016 to C001's relationships; reciprocal C001 lines added to C007 and C016. |
| 5 | "One Source Registration component" overclaimed | Reworded to "One Source Registration **capability**." Body now names the four-component bundle (C002 declaration, C003 secret, C004 registry, C005 adapter contract). Parallels the "mechanism" wording in the override principle. |
| 6 | C014 band placement (Connection-flavored name, State band) | One-line justification added inside C014's responsibility: lives in State (not Connection) because the registered entries are per-item facts the bird-dogger curates, not connection configuration. Kept the "-Registry" naming pattern (which is consistent across C004 / C006 / C014 as "registered named collection"). |
| 7 | Reasoning band intro oversimplified ("reads State") | Updated to "reads from Connection and State, writes State, hands results to the Renderer." Read-only-against-sources kept as its own standalone sentence. |
| 8 | R002 (List-Level Urgency Signals) "no new component" undefended | Expanded **C015's responsibility** to explicitly own list-level signal selection per R002, with PRD001/PRD002 obligations applying at composition time, not only render time. Map entry rewritten to name C015 as the owner (instead of a parenthetical hedge). |
| 9 | PRD001 principle overclaimed "every per-item state model" | Reworded to "every per-item **reading or produced judgment**." Body names C009 (assessment), C011 (recommendations), and C014 (contact disagreement) as first-class carriers; storage stores preserve what's written to them. **C011's responsibility** updated to explicitly carry "cannot recommend" per PRD001+PRD004 composition (matching what C009 already does). |
| 10 | Response-window computation (R003/O005) had no named owner | C013 responsibility now states the per-item response window (now − most-recent-touch time) is derived by readers, not stored; C013 anchors the derivation. Relationships tightened to say "derived response window" at both reader sites (C011 for timing recommendation, C015 for display). |
| 11 | C004 and C006 did not reciprocate C001 in their relationships | Added C001 line to each: "written by `source` (or `hunt`) declaration verbs; validates and delegates persistence to C002." Captures the precise flow (C001 → C004/C006 for validation/intent → C002 for persistence). |
| 12 | Material engineering decisions lack ADRs | **Deferred** to its own modification session. Candidates enumerated below. |

## Resulting Architecture State

- **16 components.** Edge: C001, C002, C003, C015. Connection:
  C004, C005, C006. State: C007, C010, C012, C013, C014, **C016
  (new)**. Reasoning: C008, C009, C011.
- **State band split along the per-item / per-hunt seam.** C007
  holds facts about items regardless of hunt; C016 holds answers
  to "what does this hunt currently include, used to include,
  decided not to include, and last refreshed when."
- **C007 is the identity hub.** Every State-band peer (C010,
  C012, C013, C014, C016) keys into items by C007's internal id.
- **All chase verbs that write to the State band are reciprocal.**
  C001 ↔ C007/C016/C010/C012/C013/C014 each named on both ends.
- **All deferred decisions are flagged inside responsibility
  paragraphs**, not hidden in session logs. Where a component
  will eventually own a decision that's currently open, the
  responsibility says so.

## Deferred to ADR Backfill Session

Per the intent workflow (drafting → judge → review per
modification), ADR authoring is its own session. Candidates
identified during this judge session:

**Architecture-level** (live in `docs/engineering/adrs/`):

- **ADR001 — Source/Hunt decoupling.** Alternative: a single
  combined "watch" abstraction fusing connection auth and scope.
  The decoupling is the Connection band's structural backbone.
- **ADR002 — On-demand cadence, no daemon.** Alternative:
  background daemon that polls and notifies. Trade is real — a
  daemon would let `J001-O001-R001` ("surface slipping items
  without inspection") be a true push notification rather than a
  checkpoint pull.
- **ADR003 — Read-only against sources.** Alternative: write-back
  for status sync (override the source's status when the
  bird-dogger corrects it). Cost considered: keeps the
  write-permission and notification subsystems out of the
  architecture entirely.

**Component-level** (will land alongside the components pass —
each lives under its owning component's `adrs/` folder):

- SQLite driver choice (pure-Go `modernc.org/sqlite` vs. CGO
  `mattn/go-sqlite3`) — under C007.
- Multi-source picking rule for the per-item underlying-data-
  change timestamp (max-of, per-source array, composite) — under
  C008 per Issue 3.
- TUI deferral or adoption for `birddog checkpoint` — under C015.
- Source Adapter contract shape (capabilities the Jira impl
  exposes; what a future source type must implement) — under C005.

## Pressure Points Flagged for Future Judges

Noticed during this session but not in scope to resolve here:

- **C004 and C006 as resolvers vs. stores.** Issue 11 made the
  flow explicit (C001 → C004/C006 → C002), but a future judge
  could push on whether C004 and C006 are independent stores in
  their own right, or just validating resolvers over C002. The
  responsibility paragraphs read "Holds the declared set of
  sources" / "Holds the declared hunts," which is ambiguous about
  in-memory vs. persisted state.
- **C001 chase-action dispatch breadth.** C001's relationships
  show it dispatches to six chase targets (C008–C013) plus the
  two State-band stores (C007, C016) added in Issue 4. The
  components pass might benefit from a Chase Orchestrator
  component between C001 and the chase targets, especially if
  multi-store transactional invariants (e.g., manual item capture
  writing to both C007 and C016 atomically) need explicit owning.
- **C015 composition-time obligations** (Issue 8). C015 now
  carries PRD001/PRD002 + R002 signal-selection responsibilities
  at composition time. If this grows further during the
  components pass, a separate List Composer component might earn
  its keep.

## Suggested Next Steps

1. **ADR backfill session(s)** on the three architecture-level
   ADR candidates (ADR001 source/hunt decoupling, ADR002
   no-daemon cadence, ADR003 read-only sources). Treat as one or
   three modification sessions per the intent workflow. Each
   needs Context / Options / Decision / Consequences shaped like
   the PRDs.
2. **Components pass** on `docs/engineering/components/
   C<NNN>-<name>.md`. Updated likely-candidates list:
   - Simple template: C002, C003, C006, C012, C013, C014, C015,
     **C016 (new)**.
   - Nested template (will spawn ADRs early): C005 Source
     Adapter, C007 Item Store, C008 Refresh Engine, C009
     Assessment Engine, C011 Synthesis Engine.
3. **First component-level ADRs** likely to land alongside the
   components pass — as listed under "Deferred to ADR Backfill
   Session" above.

## Open Questions Carried Forward

Not blocking — live since the previous session:

- `bird-dogger` (product term) vs. `birddog` (binary name)
  distinction. Both still appear; probably fine.

New from this session:

- Whether C004 and C006 should be restated as resolvers (no
  independent storage) or as stores in their own right. Issue 11
  made the flow explicit; the noun-vs-verb framing is still
  worth a future judge's attention.
- Whether the engineering layer needs a Chase Orchestrator
  component to own multi-store write transactions (Issue 4's
  manual-capture flow is the cleanest example: C001 →
  C007 + C016 in one transaction).
