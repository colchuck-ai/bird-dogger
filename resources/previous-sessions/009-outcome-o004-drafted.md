# Session 009 — Drafted outcome O004 Triage Speed

## Goal

Use the `intent` skill to draft the next outcome document under J001.
O004 Triage Speed was the standing next pick from sessions 007 and
008 — chase-loop step three (Detect → Cover → Triage → Escalate), and
more independent of the remaining outcomes than O005 (which S002
flagged as packing both targeting and timing).

## Context loaded

- `docs/product/README.md` — current product root (with the
  "checkpoint" definition added in S008).
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  sibling; slip indicators are inputs to triage.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  triage operates on items already in the active set.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; status confidence is an input to triage.
- `resources/previous-sessions/001` through `008` — locked
  decisions, deferred risks, standing fragilities, the S008 process
  slip on CR-during-create.
- `resources/bird-dogging/bird-dogging-overview.md` — "blockers and
  dependencies" called out as part of bird-dogging core practice,
  which seeded RSK005.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/context-tracing.md` (Outcome trace), `references/
  workflows.md` (three-session model, create has no CR),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no
record history needed for a create.

## Workspace path mismatch (still standing from sessions 002–008)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Wrote against the real
path. Worth fixing the workspace mapping.

## Outcome selection

Pre-decided by user prompt and by S007/S008's standing
recommendation. No alternative survey this session.

## Outcome framing decisions

Drew the outcome line at "given items are in coverage (O002) with
slip detection (O001) and trustworthy status (O003), how fast can
the bird-dogger figure out **which item needs them next?**"

Triage Speed is the *synthesizer* outcome. It consumes the upstream
outcomes' signals — slip status, status confidence/freshness, due
dates, blocker age, dependencies — and the failure modes here are
about turning those signals into a "who needs me now" call without
the bird-dogger re-deriving the priority of every item by hand.

Boundary against siblings:

- **vs O001 Slip Detection Lag.** O001 surfaces *that* something is
  slipping. O004 picks *which* slipping/stale/blocked item to act
  on first. Slip status is an input.
- **vs O002 Coverage.** O002 keeps items on the list. O004 assumes
  they are on the list and asks which to act on first.
- **vs O003 Status Fidelity.** O003 makes the status trustworthy.
  O004 weights status as one of the urgency signals.
- **vs O005 Escalation Quality.** O005 starts after triage — given
  "this item needs me now," escalate well. O004 stops at the
  identification.

## Risk framing decisions

Five risks, each customer-centric (the bird-dogger's triage
friction, not system internals), solution-free, and closing on the
outcome statement clause `...increasing the time to identify which
items most need intervention right now.` (Pattern preempted from
S004's late fix on O003.)

- **RSK001 No Urgency Ranking.** Items are flat-listed or only
  binary-flagged ("needs attention" vs "ok"). Bird-dogger has to
  evaluate every flagged item to pick what comes first. Folded an
  earlier candidate "Undifferentiated Urgency" (every slipping item
  flagged identically) into this risk — same failure mode at a
  different granularity.
- **RSK002 Buried Urgency Signals.** Urgency-bearing facts live in
  detail views, not at the list level. Parallel structure to
  O001-RSK002 (Buried Slip Signal), but the *object* differs: O001
  is about a slip indicator being mixed in with healthy items; O004
  is about the *full set* of urgency facts (slip + freshness +
  due-date proximity + blocker age) needing to be visible at the
  list level so triage doesn't require drilling in.
- **RSK003 Undefined Intervention Threshold.** Product shows
  signals, never commits to when they collectively mean "needs
  intervention now." Bird-dogger has to construct the threshold
  themselves each checkpoint. This is the genuinely *new* failure
  mode for O004; not visible from upstream outcomes.
- **RSK004 Triage Memory Loss.** Bird-dogger's prior triage
  decisions ("watching," "deferred," "intervened, awaiting reply")
  not carried forward; every checkpoint starts from scratch on
  unchanged items. The risk is not about losing the *items* (that's
  O002 territory) — it's about losing the *triage state* on those
  items.
- **RSK005 Hidden Item Dependencies.** A blocked-on-B relationships
  not surfaced at the list level; bird-dogger may pick A as urgent
  without seeing that A cannot move until B moves. Seeded by
  `bird-dogging-overview.md`'s "blockers and dependencies" core
  responsibility.

## Boundary calls worth flagging for the judge

- **RSK002 (O004) vs O001-RSK002 (Buried Slip Signal).** Both about
  signals being visually buried. Distinct *objects*: O001 is the
  slip signal alone (slipping items mixed with healthy ones); O004
  is the full urgency-signal bundle (slip + freshness + due-date +
  blocker-age) at list level for triage. Judge may probe.
- **RSK004 (Triage Memory Loss) vs O002 (Coverage).** Could read as
  "items dropping out of view between checkpoints" which is
  O002-RSK003 (List Drop-Off). Distinct: O002-RSK003 is the *item*
  leaving the view; O004-RSK004 is the *triage decision* on the
  item being forgotten while the item is still on the list. Same
  resolution pattern as O002-RSK003 vs O003-RSK003: shared
  surface, distinct failure modes.
- **RSK005 (Hidden Item Dependencies) fold into RSK002 (Buried
  Urgency Signals).** Tempting — dependencies could be just "another
  urgency-bearing fact." Argued distinct: dependencies are
  *relationships between items*, not item-level urgency
  attributes. R005 mitigates by surfacing a relationship; R002
  mitigates by surfacing item-level signals. Judge may try to fold.

## Requirement framing decisions

Five requirements, each tied to ≥1 risk. All behavioral.

- **R001 Urgency Ranking.** Direct mitigation of RSK001. Constrained
  to "with enough granularity to differentiate" to preempt the
  binary-ranking degenerate case the risk names.
- **R002 List-Level Urgency Signals.** Direct mitigation of RSK002.
  Names the surface (list-level) but not the mechanism.
- **R003 Intervention Recommendation.** The most product-shaping
  requirement here. The product must commit to a notion of "needs
  intervention now" and apply it consistently, rather than leaving
  the synthesis entirely to the bird-dogger. Doubles as a partial
  mitigation of RSK001 — if the recommendation reduces the set of
  items the bird-dogger must consider, urgency ranking matters
  less on the items below the threshold.
- **R004 Triage State Carry-Forward.** Records the bird-dogger's
  prior triage decision and resurfaces it at the next checkpoint
  *alongside what has changed.* The "alongside what has changed"
  clause is doing real work: it tells the product *what state* to
  carry, but pairs it with delta-since so the bird-dogger doesn't
  blindly trust a stale "watching" decision when underlying facts
  have moved.
- **R005 Inter-Item Dependency Surfacing.** Mitigates RSK005 by
  putting dependency relationships at the list level. Stops short
  of saying *how* dependencies are inferred or recorded — that is
  engineering.

Risk-Requirement Map:

- RSK001 → R001, R003
- RSK002 → R002
- RSK003 → R003
- RSK004 → R004
- RSK005 → R005

R003 carries two risks (RSK001 partial, RSK003 direct). Defensible:
RSK001 is "no ordering" and RSK003 is "no threshold"; R003 supplies
a threshold/recommendation, which both ranks-by-cutoff (RSK001
partial) and resolves the threshold question (RSK003 direct). Pure
ranking still belongs to R001.

All other mitigations are single. Pattern matches O002-RSK005 → R006
(single mitigation, documented fragility, not a defect).

## What got written

`docs/product/outcomes/J001-O004-triage-speed/README.md` — full
outcome document. Outcome statement lifted verbatim from product
root for narrative coherence (same move as S3, S5, S7). Five risks,
five requirements, risk-requirement map.

No Change Record — this is a create, per workflows.md (and the
S008 process slip lesson).

## Coverage check

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All five risk impact clauses close on the outcome statement
  (`...increasing the time to identify which items most need
  intervention right now.`).
- **R003 carries 2 risks** (RSK001 partial + RSK003 direct). Lower
  load than O001-R001's 3-risk position pre-judge or O002-R001's
  3-risk position pre-judge. Defensible.
- **RSK002, RSK003, RSK004, RSK005 each have single-mitigation**
  (R002, R003, R004, R005 respectively). Standing fragility,
  parallel to O001-RSK002 / O002-RSK005 / O003-RSK002 patterns. Not
  a defect.

## Deferred (so we don't lose it)

- **R003 (Intervention Recommendation) solution-talk pressure.**
  "Synthesize urgency signals into a recommendation" implies a
  decisioning mechanism. Argued: *what* (commit to a notion of
  needs-intervention-now and apply it consistently); engineering
  layer decides *how* (heuristic, scoring model, local-LLM, rule
  set). Same shape as O001-R001 and O002-R003. Judge will probe.
- **R004 "alongside what has changed" clause.** Borders on
  prescriptive. Argued: it specifies *what* the bird-dogger sees,
  not *how* the diff is computed. Judge may probe.
- **R005 (Inter-Item Dependency Surfacing) fold-into-R002 pressure.**
  Dependencies as "another urgency-bearing fact" would collapse
  R005 into R002. Argued distinct: relationships vs item-level
  attributes. Judge will probe.
- **RSK005 (Hidden Item Dependencies) standalone-ness pressure.**
  Could fold into RSK002 (Buried Urgency Signals) under a generous
  reading of "facts." Same argument as the R005-vs-R002 case.
- **R003 doubling on RSK001 + RSK003.** Argued distinct mitigation
  paths (ranking-by-cutoff vs threshold). Judge may try to pull R003
  off RSK001 if R001 alone is enough.
- **RSK004 (Triage Memory Loss) vs O002-RSK003 (List Drop-Off)
  boundary.** Standing flag — same resolution pattern as
  O002-RSK003 vs O003-RSK003 should apply, but judge should
  confirm.
- **RSK002 (O004) vs O001-RSK002 boundary.** Standing flag — same
  pattern; different objects.
- **R005 (O004) fragility under single-risk coverage.** New this
  session.
- **R002 (O004) fragility under single-risk coverage.** New this
  session.
- **R004 (O004) fragility under single-risk coverage.** New this
  session.
- **Engineering concerns** from sessions 001 / 002 now sharpened by
  O004:
  - **Pluggable inputs** — R002 (List-Level Urgency Signals)
    consumes slip indicators, freshness, due-date proximity,
    blocker age. Adds to the pluggable-input pressure already
    coming from O001-R007, O002-R002 / R005 / R007.
  - **Local-LLM decisioning** — R001 (Urgency Ranking) and R003
    (Intervention Recommendation) are the strongest signal yet
    that some kind of decisioning component is implied at the
    architecture stage.
  - **Decision cache keyed by issue-data hash** — R004 (Triage
    State Carry-Forward) reads directly onto this; if cached
    triage state is keyed by the issue's data hash, "alongside
    what has changed" follows for free.
  - **Pluggable outputs** — not advanced by O004.
- **R007 cross-outcome duplication** (Source Registration in O001
  and O002, identical wording). Still standing from S008. O004
  doesn't reuse it.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from S004. Distinct R005
  there — naming clash with this session's R005, unrelated.
- **O001-RSK002 single-mitigation fragility** (R001 only) — still
  standing from S006.
- **Vim swap file** `.README.md.swp` in the O003 folder — flagged
  in sessions 004 / 005 / 006 / 007 / 008; not touched here.
  Author should reconcile.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these changes (and any uncommitted prior-session
   edits if still pending) so an independent reviewer can run
   session 010 (judge pass), then session 011 (review).
2. Judge pass against the Outcome / Risk / Requirement rubrics.
   Likely-fruitful prompts:
   - RSK005 / R005 vs RSK002 / R002 fold attempt — distinct
     relationships vs item-level attributes, or one risk?
   - R003 (Intervention Recommendation) solution-talk pressure.
   - R004 "alongside what has changed" prescriptiveness check.
   - R003 doubling on RSK001 + RSK003 — is R001 alone enough for
     RSK001?
   - RSK004 vs O002-RSK003 boundary.
   - RSK002 (O004) vs O001-RSK002 boundary.
   - Single-mitigation fragilities on RSK002 / RSK004 / RSK005.
3. After judge + review on O004, draft **O005 Escalation Quality**
   — the last outcome. S002's flag about O005 packing both
   targeting and timing still stands; may need to split or argue
   the joint framing during drafting.
