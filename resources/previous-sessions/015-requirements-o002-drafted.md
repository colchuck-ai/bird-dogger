# Session 015 — Drafted requirement documents for O002 Coverage

## Goal

Use the `intent` skill to draft the per-requirement documents for
the next important outcome under J001. S014's recommendation was
PDR-session-first (lift "cannot assess" posture and R007 cross-
outcome capability to product-root PDRs before drafting more
requirements). User chose to push down into O002 requirements
instead. The PDR-first deferral is now standing more loudly.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- `docs/product/outcomes/J001-O002-coverage/README.md` — outcome
  under drafting; requirement statements lifted verbatim.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md`
  — sibling; shares R007 (Source Registration) wording. O001-R006
  named in O002-R006 deps for source-availability surfacing
  (deferred up, not duplicated).
- `docs/product/outcomes/J001-O003-status-fidelity/README.md`,
  `J001-O004-triage-speed/README.md`,
  `J001-O005-escalation-quality/README.md` — siblings; consume
  the active set this outcome defines.
- One sample O001 requirement file
  (`J001-O001-R001-proactive-slip-surfacing.md`) and the R007
  doc — for in-doc style + R007 cross-outcome duplication
  precedent.
- `resources/previous-sessions/013-requirements-o001-drafted.md`
  — drafting precedent (statements verbatim, simple template,
  no CR, no Records, "cannot assess" recurrence).
- `resources/previous-sessions/014-requirements-o001-judged.md`
  — most recent judge precedent (R001 dependency vs adjacency,
  R007 PDR-candidate, R008 de-registration gap in O002 standing
  as (a)-path).
- `intent` skill: `SKILL.md` (Requirement rubric),
  `references/context-tracing.md` (Requirement self/siblings/
  cousins), `references/workflows.md` (drafting inside create
  cycle, no CR), `assets/templates/requirement-simple.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance for a
create cycle. Did not pre-load O001's other R-docs in full —
only deps were named cross-outcome, not internalized.

## Workspace path mismatch — resolved late in session, but not as prior sessions assumed

Prior sessions (S002–S014) flagged this as Cursor opening a
workspace path that resolved via symlink to the real repo. That
assumption was wrong. After the user renamed `colchuck-ai/
bird-dogging-skill` → `colchuck-ai/bird-dogger` mid-session, two
separate physical directories were exposed:

- `colchuck-ai/bird-dogger/` — the real git repo (with `.git`,
  `LICENSE`, all prior O001 work, all session digests through
  014).
- `jomadu/bird-dog-skill/` — a sparse scratch dir, no `.git`,
  no `LICENSE`. Not a symlink. Held only this session's writes
  (8 files: 7 R-docs + this digest) and nothing else.

So during this session's drafting:
- Read tool calls against the workspace path failed (as in S013/
  S014), fell back to shell `cat` which ran from the real repo
  path (shell `pwd` was the colchuck path).
- Write tool calls landed in the sparse `jomadu/bird-dog-skill/`
  dir, divergent from the real repo.

After the rename was reported, files were relocated:
`cp` from `jomadu/bird-dog-skill/` → `colchuck-ai/bird-dogger/`
at matching paths; `jomadu/bird-dog-skill/` removed. `git status`
on `bird-dogger/` confirms the 7 R-docs + this digest as the
only untracked changes.

**Standing for next session:** the Cursor workspace path should
be repointed at `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogger` (the renamed real repo) so reads and writes line up
without a manual relocation step. Until that's done, the same
divergence will reappear on any future session that writes
through tool calls.

## Outcome selection

User asked for "the next important outcome." Pre-survey across
the four un-drafted outcomes (O002–O005) against two lenses:

- **O002 Coverage** — feeds every downstream outcome (if items
  aren't in coverage, slip/status/triage/escalation never see
  them); shares R007 with O001 (one capability surfaces under
  two outcomes); R008 de-registration gap from S014 (a)-path
  flagged to land here. Strongest coupling.
- **O003 Status Fidelity** — boundary heavy (R002 already
  surfaced as a per-item assessment pattern parallel to
  O001-R002).
- **O004 Triage Speed** — consumes O001 slip set + O002 active
  set as inputs; better drafted after both upstream outcomes
  exist as requirements.
- **O005 Escalation Quality** — consumes O001 + O004; latest in
  the chain.

Recommended **O002** for coupling to O001 + shared R007 + the
standing R008 de-registration gap. User confirmed via
AskQuestion. No alternative survey beyond the four pre-presented
options.

## Template choice

Simple template for all 7. None of the O002 requirements carry a
PDR, CR, or other attached material yet. Nested would create
empty `pdrs/` and `crs/` subdirectories. Same precedent as O001
(S013).

## Requirement framing decisions

Statement lines lifted verbatim from the outcome README's
requirements list — no rewording. Same trace-preservation
pattern as S013.

R007 was deliberately mirrored from the O001-R007 doc rather than
re-drafted from scratch. Detail section carries the same in-doc
duplication acknowledgement ("identical wording to J001-O001-
R007; intentional; single capability; standing cross-outcome
flag for engineering layer"). The flag is now doubled (two
docs, identical wording) — PDR pressure is louder, not weaker.

Cross-cutting moves applied to all 7:

- **"Cannot assess" / coverage-over-precision posture used
  implicitly.** R001 (source-detached items still shown), R004
  (drop-offs surfaced explicitly), R006 (items retained when
  source unreachable, flagged stale). Same posture S014 flagged
  as PDR-candidate. Now used twice without the PDR landing.
- **Acceptance criteria stay solution-free.** No UI nouns, no
  channel choices, no thresholds. Verifiable but behavioral.
  S013/S014/S010 precedent.
- **Edge cases bias toward fail-safe defaults.** Every doc
  carries at least one edge case where "we don't know" forces
  explicit surfacing rather than silent omission (R001 source-
  detached, R002 dedup-on-second-capture, R003 unsupported
  candidate source, R004 bulk-archive sweep, R005 ambiguous
  migration, R006 source unreachable at refresh time, R007
  unsupported source type).
- **Source de-registration was NOT added as R008.** S014's
  (a)-path recommended R008 in O002 alongside R007. The outcome
  README lists R001–R007 only; not in scope to invent an R008
  outside the README. Gap stands; promoted in Deferred.

Per-requirement notes:

- **R001 (Coverage Set Visibility).** Hub for the set-of-sources
  / set-of-items pair. Deps: R003, R006, R007. Risk: R001 ↔ R003
  cycle in the dep graph (see Coverage check); same dependency-
  vs-adjacency tension S014 resolved on O001-R001/R002. Judge
  will probe.
- **R002 (Manual Item Capture).** First-class membership in the
  active set explicitly committed — manual items participate in
  slip/status/triage/escalation on equal footing with source-
  backed items. R002 ↔ R005 reciprocal dep on manual-meets-
  source-record link.
- **R003 (Coverage Gap Discovery).** Two candidate kinds
  (sources and items) framed as decisions, not silent additions.
  "Rejected candidates remembered" introduces an implicit
  triage-memory shape parallel to O004-R004. Judge may probe
  whether this should defer to O004's mechanism rather than
  re-state.
- **R004 (Drop-Off Surfacing).** "Keep in coverage in one
  action" in AC #3 is mildly prescriptive — parallel to O001-
  R004's finite-state-set call in S014 (judge held). Reviewer
  may soften to "without re-derivation" or similar.
- **R005 (Item Migration Linking).** "Ambiguous migration"
  edge case left unparameterized at requirement layer — same
  shape as O001-R005 silence-window in S014 (judge held).
  Engineering-layer concern: linking heuristics. Reads onto
  the same decisioning component flag as O001-R001 and O004.
- **R006 (Active Set Freshness).** Explicitly defers source-
  unavailability surfacing to O001-R006 (cross-outcome). Edge
  cases force partial-refresh honesty (some sources reachable,
  some not, set reflects what was reachable). Reads onto the
  per-item source-availability pattern flagged in S013/S014 +
  Refresh interval / push-vs-pull deferred to engineering.
- **R007 (Source Registration).** Wording identical to O001-
  R007. Detail names the cross-outcome duplication; deps cite
  O001-R007 explicitly. Cross-outcome edge in the graph.

## Boundary calls worth flagging for the judge

- **R001 ↔ R003 dependency cycle.** R001 deps R003 ("gap
  discovery surfaces what is not in the watched/active sets and
  depends on those sets being defined"); R003 deps R001 ("gap
  discovery is defined relative to the watched/active sets
  shown by R001"). Both read as adjacency, not consumption.
  Likely judge resolution: drop one direction (probably R001's
  dep on R003, matching the O001 R001/R002 precedent in S014).
- **R001 ↔ R007 dependency cycle.** R001 deps R007 ("registration
  creates the entries this view lists"); R007 deps R001
  ("registered sources appear in the watched-sources view").
  Same adjacency tension. Likely judge: drop R007's dep on
  R001.
- **R002 ↔ R005 dependency cycle.** R002 deps R005 (manual item
  later appearing in a source); R005 deps R002 (manual captures
  later linked to a source record). Symmetric. Likely judge:
  collapse to one direction or accept as boundary-bidirectional
  with the rationale stated.
- **R003 "rejected candidates remembered" vs O004-R004.** Same
  carry-forward shape under a different outcome. Reviewer-
  fruitful — is this an O002 capability or an O004 one applied
  to coverage?
- **R004 "in one action" wording.** Mildly prescriptive; may
  soften.
- **R005 ambiguous-migration verifiability.** Same shape as
  O001-R005 silence-window; pattern previously held by judge in
  S014. Likely a hold here too.
- **R006 refresh trigger unspecified.** Engineering-layer
  deferral. Reviewer may probe.
- **R007 in-doc duplication note (doubled).** S014 deferred the
  resolution to a PDR session. Now the rationale lives in two
  docs identically; the PDR pressure is the strongest standing
  flag this session.
- **R008 de-registration gap.** S014's (a)-path explicitly named
  O002 as the home. Not added. The outcome README does not
  list an R008. Either (a) update the outcome README to add
  R008 with a fresh draft, (b) accept the gap and reframe O001-
  R006's retired-source edge case to point at a future
  capability, or (c) leave standing until a separate session
  addresses it. This session took path (c) by default.

## What got written

Seven files under
`docs/product/outcomes/J001-O002-coverage/requirements/`:

- `J001-O002-R001-coverage-set-visibility.md`
- `J001-O002-R002-manual-item-capture.md`
- `J001-O002-R003-coverage-gap-discovery.md`
- `J001-O002-R004-drop-off-surfacing.md`
- `J001-O002-R005-item-migration-linking.md`
- `J001-O002-R006-active-set-freshness.md`
- `J001-O002-R007-source-registration.md`

All simple template. Each: lifted statement, Detail, Edge Cases,
Examples (one scenario each), Acceptance Criteria (3–4
conditions each), Dependencies (intra-outcome plus the two
named cross-outcome edges on R002 → O001-R003, R006 → O001-R006,
and R007 → O001-R007).

No Change Record — drafting session in a create cycle (S005 /
S007 / S009 / S011 / S013 precedent and `workflows.md`).

## Coverage check

- Every requirement listed in the O002 README has a per-
  requirement document: ✓ (7 of 7).
- Every per-requirement document's statement matches the outcome
  README verbatim: ✓ (trace-preserving).
- R007 wording byte-equivalent to O001-R007 statement: ✓.
- Dependency graph: connected, **but contains three cycles**
  (R001 ↔ R003, R001 ↔ R007, R002 ↔ R005). Material judge
  finding; same shape S014 resolved on R001 ↔ R002 in O001 by
  dropping one direction. Carried forward for judge.
- Cross-outcome dep edges asserted (R002 → O001-R003, R006 →
  O001-R006, R007 → O001-R007) are consistent with O001's
  drafted state.
- Risk–requirement map in the O002 README is unchanged and
  intact: RSK001 → R002; RSK002 → R001, R003, R007; RSK003 →
  R004; RSK004 → R005; RSK005 → R006.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk; every
  risk impact clause closes on the outcome statement
  ("forgotten between checkpoints").
- **Sibling (intra-outcome):** R001 is the visibility hub; R007
  is the registration hub; R003 is the discovery hub. Three
  cycles in the dep graph (see Coverage check).
- **Cousin (cross-outcome) — sharpened this session:**
  - **O002-R007 ≡ O001-R007.** Source registration is one
    capability surfaced under two outcomes; now duplicated in
    two requirement docs with identical wording. PDR-candidate
    pressure doubled.
  - **O002-R006 defers to O001-R006** for source-availability
    surfacing during refresh. Same per-item-availability
    pattern S014 flagged.
  - **O002-R005 ↔ O001 active set assumptions.** Manual items
    captured under O002-R002 must remain assessable under
    O001-R003 (Expected Trajectory) — explicitly named in
    R002's deps.
  - **O002-R003 "rejected candidates remembered" ↔ O004-R004
    (Triage State Carry-Forward).** Same carry-forward shape;
    reviewer-fruitful.
  - **No-owner-at-all seam between O002 and O005** — still
    standing from S012; R002's "manual item with no owner
    recorded" edge case touches this seam without resolving it
    (deferred to O005 / engineering).

## Deferred (so we don't lose it)

New this session:

- **R001/R003/R007 dependency cycles.** Three of them; likely
  judge prunes. Same shape S014 resolved on R001 ↔ R002.
- **R002 ↔ R005 reciprocal dep.** Worth a single-direction or
  explicit-boundary call.
- **R003 "rejected candidates remembered" boundary vs O004-R004.**
  Whose carry-forward owns this?
- **R004 "in one action" wording softness.**
- **R005 ambiguous-migration verifiability without a number.**
  Same shape as O001-R005 silence-window (held in S014).
- **R006 refresh trigger (poll vs push) — engineering deferral.**
- **R008 de-registration in O002 — still not added.** S014's
  (a)-path remains unmet; this session left the gap standing
  because the outcome README does not list R008. Needs either a
  README update + draft pass or an explicit accept.
- **R007 in-doc duplication — doubled.** Two docs now carry the
  same paragraph. PDR pressure is the strongest standing flag.

Promoted to PDR-candidates (deferred since S014; pressure
increased this session):

- **"Cannot assess" / coverage-over-precision posture.** Now
  used in O001-R001/R004/R006 *and* O002-R001/R004/R006. Six
  separate per-requirement docs assert the posture; the PDR
  should land before further requirement drafts repeat it.
- **R007 Source Registration as cross-outcome capability.** Now
  literally duplicated wording in two docs.

Carried forward:

- **O001-RSK002 single-mitigation fragility** (R001 only).
  Standing from S006. Unchanged.
- **R005 fragility under O003** (lone preventive). Standing from
  S004. Three R005s in the tree now (O001, O002, O005); content
  distinct, naming coincidence.
- **No-owner-at-all seam between O002 and O005.** Standing from
  S012; touched by O002-R002 edge case this session, not
  resolved.
- **Engineering concerns** (sharpened):
  - **Pluggable inputs** — R007 (now in two outcomes) widens
    input set; refresh cycle (R006) consumes the same
    registration.
  - **Decisioning component** — R005's migration-link heuristic
    joins O001-R001/R005, O004-R001/R003, O005-R004 as
    decisioning-component pressure.
  - **Per-item state with freshness** — R006 "last refresh"
    timestamp + per-source partial-availability granularity
    extends the per-item cache pattern S013/S014 flagged.
  - **Per-item source-availability attribution** — R006 defers
    to O001-R006; the per-item shape (not just per-source) is
    now relevant in two outcomes.
  - **History / event store** — R002 dedup, R004 drop-off
    history, R005 migration history, R003 rejected-candidate
    memory all reach for the same audit/event store flagged in
    S014 as engineering-layer.
- **Workspace path mismatch** in Cursor. Real repo path is now
  `colchuck-ai/bird-dogger` (renamed from `bird-dogging-skill`
  this session). Workspace still points at
  `jomadu/bird-dog-skill` and needs repointing; see the
  Workspace path mismatch section above.

Cleared:

- (none) — vim swap files remain cleared per S011.

## Recommended next step

1. **Resolve the PDR-first deferral.** S014 already recommended
   PDR session before more requirement drafts. The pressure has
   doubled. Two PDRs worth opening before O003/O004/O005
   requirement drafts:
   - "Cannot assess" / coverage-over-precision posture (six
     per-requirement repetitions now).
   - R007 Source Registration as one cross-outcome capability
     (now literally duplicated in two docs).
2. **Judge session 016** on the seven O002 requirement docs
   against the Requirement rubric. Likely-fruitful prompts
   (in priority order):
   - Three dep cycles (R001 ↔ R003, R001 ↔ R007, R002 ↔ R005)
     — prune to one direction or accept as boundary.
   - R003 "rejected candidates remembered" vs O004-R004 — own
     here or defer there?
   - R008 de-registration gap — add as R008 (update outcome
     README + draft) or formally accept the gap.
   - R002 "manual item with no owner recorded" — touches the
     O002 ↔ O005 no-owner seam; resolve or defer.
   - R004 "in one action" wording softness.
   - R005 ambiguous-migration verifiability without a number.
   - R006 refresh trigger (poll vs push) — engineering or
     requirement?
3. **Then PR + review** the judged O002 docs against the
   Requirement rubric (session 017 shape).
4. **Decision point after review on O002:**
   - Do PDR session(s) before O003/O004/O005 requirement
     drafts (strongly recommended — the cross-cutting flags
     have already started repeating).
   - Or parallelize O003/O004/O005 requirement drafts across
     reviewers and accept the rework that the eventual PDRs
     will force.
