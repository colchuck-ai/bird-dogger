# Session 012 — Judged outcome O005 Escalation Quality

## Goal

Run the judge pass on
`docs/product/outcomes/J001-O005-escalation-quality/README.md` (drafted
in session 011) against the Outcome / Risk / Requirement rubrics in
the `intent` skill, before going to review.

## Context loaded

- `docs/product/outcomes/J001-O005-escalation-quality/README.md` — the
  draft under judgment.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  sibling; slip status is an input to R004's timing recommendation;
  RSK005 (Source Blind Spot) is the structural precedent for keeping
  RSK001's two-cause fold unified.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  owner-identification boundary; "no-owner-at-all" seam.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; RSK002 boundary check against O003-RSK002 (different
  objects — owner response window vs status data staleness).
- `docs/product/outcomes/J001-O004-triage-speed/README.md` — sibling
  immediately upstream; triage ends where escalation begins; R004 /
  RSK004 / R005 all run parallel to O004 patterns.
- `resources/previous-sessions/004-outcome-o003-judged.md`,
  `006-outcome-o001-judged.md`, `008-outcome-o002-judged.md`,
  `010-outcome-o004-judged.md` — prior judge precedent (clean-vs-
  edits patterns, fold-vs-split resolutions, intermediate-clause
  precedent, single-mitigation fragility framing, CR-or-not).
- `resources/previous-sessions/011-outcome-o005-drafted.md` — the
  draft session's standing judge prompts (eight of them) and the
  S002 joint-framing flag.
- `resources/bird-dogging/bird-dogging-overview.md` — escalation as a
  core responsibility; cadence step 4.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/context-tracing.md` (Outcome trace),
  `references/workflows.md` (judge inside create cycle has no CR),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — judge on
a create cycle.

## Workspace path mismatch (still standing from sessions 002–011)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Worked from the real
path. Worth fixing the workspace mapping.

## Verdict

Clean. No material edits. All eight standing judge prompts from S011
resolve in the draft's favor on precedent. Coverage and trace check
out. One soft observation on RSK002 framing (intermediate "eroding
credibility" clause) left standing — defensible against sibling
precedent. Second clean judge pass after S006; S004 / S008 / S010
all required edits.

## Standing judge prompts from S011 — resolutions

1. **RSK001 internal fold (stale-owner + missing-chain) — keep
   folded.** Direct precedent: O001-RSK005 (Source Blind Spot) is
   single failure mode ("slip invisible") with two causes ("outage
   or because the source has not been integrated"); S006 kept it
   unified. RSK001 here is the same shape: single failure
   ("escalation reaches someone who cannot move the item") with two
   causes ("recorded owner is no longer the right person, or because
   no escalation chain above the owner is recorded"). Parallel
   construction; same resolution. Mitigations split cleanly (R001 →
   chain, R002 → owner currency) which raised the fold pressure,
   but the customer-side failure mode is unitary.
2. **R004 solution-talk pressure — pass.** "Synthesize signals into
   a recommendation" is the *what*. Same shape and defense as
   O004-R003 (upheld in S010). The explicit signal list (slip
   status, intervention window, response window) is a testability
   win and can be broadened via update CR if new signals emerge —
   not a constraint defect.
3. **R004 doubling on RSK002 + RSK003 — keep both links.** RSK002
   is "too soon," RSK003 is "too late." R003 supplies the *fact*
   of the response window; R004 commits the *threshold* that
   defines "fair." Without R004, the bird-dogger derives "fair"
   from scratch each checkpoint — same shape as O004-RSK003
   (Undefined Intervention Threshold). R004 has a distinct
   RSK002-mitigation role beyond R003. Mirrors O004-R003's
   RSK001+RSK003 doubling, which S010 upheld with the same defense.
4. **RSK002 (O005) vs O003-RSK002 boundary — keep distinct.**
   Different *objects*: O005-RSK002 = owner's response window after
   the bird-dogger's last touch; O003-RSK002 = item's status data
   going stale. Different clocks, different things going stale. No
   fold pressure.
5. **RSK004 (O005) vs O004-RSK004 boundary — keep distinct.** Same
   shape (state-loss-between-checkpoints), different objects
   (escalation history vs triage decision). Mirrors the
   O004-RSK004 vs O002-RSK003 pattern resolved in S010.
6. **Channel-mismatch absence — pass.** "Right person at right
   time" absorbs it; right-owner-wrong-channel lands the same as
   RSK001 (Misdirected Target) in customer experience (escalation
   reaches someone who cannot act on it now). R005 records channel,
   so the data is captured for a future channel-mismatch risk if
   warranted.
7. **No-owner-at-all absence — pass with flag.** Defensible
   upstream territory: O002 keeps items on the list with an owner
   recorded; O005 presumes a target exists. Thin seam where "item
   on the list with no recorded owner" is uncovered — coverage
   problem before it's an escalation-quality problem. Standing flag
   for review.
8. **Single-mitigation fragility on RSK003 (R004 only) and RSK004
   (R005 only) — flag, not defect.** Parallel to O001-RSK002 /
   O002-RSK005 / O003-R005 / O004-RSK002 / RSK004 / RSK005.
   Documented standing cost.

## New issues the judge surfaced

**RSK002's "eroding credibility" intermediate clause — kept,
asymmetric but defensible.** RSK002 is the only risk in O005 that
routes through a second-order consequence (credibility erosion)
before closing on the outcome; the other three close direct. Two
reads:

- (a) tighten by dropping the intermediate; align with siblings.
- (b) keep — credibility erosion is *the* failure mode for
  premature escalation. Without it, "you escalated too early"
  reads thin against the outcome's "right time" clause (the owner
  *could* still act on an early escalation; what hurts is the
  unjustified timing eroding trust in future ones).

Intermediate mechanism clauses are precedented — O001-RSK002
("requiring the bird-dogger to search before recognizing the
slip"), O004-RSK002 ("must inspect each item individually before
recognizing what is urgent"). RSK002's clause does parallel work —
names the meaningful cost between condition and outcome. No edit.
Reviewer-fruitful probe.

**R001 naming — minor.** "Escalation Target" reads loosely (the
owner is the *intervention* target; escalation targets are above
the owner in the chain). But R001's purpose is to surface the
candidate targets (owner + chain), so the noun-noun naming holds.
Parallel to O003-R001 (Status Basis). No edit.

## What got edited

Nothing. Draft stands as-is.

No Change Record — judge inside a create cycle (S006 / S008
precedent, `workflows.md`).

## Coverage check

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All four risk impact clauses close on the outcome statement
  (`...reducing confidence that escalations land with the right
  person at the right time.`).
- **R004 carries 2 risks** (RSK002 + RSK003). Defensible per S010
  precedent on O004-R003's RSK001 + RSK003 doubling.
- **RSK003, RSK004 each single-mitigation** (R004 and R005
  respectively). Standing fragility, parallel pattern.

## Vertical & horizontal trace

- Outcome statement lifted verbatim from product root — no drift.
- Outcome serves J001 as chase-loop step 4 (tail); consumes O004's
  pick of which item needs intervention now.
- Sibling boundaries (O001 / O002 / O003 / O004) hold — see prompts
  4, 5, 7.
- Cousin pressure on the engineering layer (sharpened this session):
  - **Pluggable inputs** — R001 (target / chain data), R003
    ("bird-dogger's last touch" knowledge). Adds to O001-R007,
    O002-R002 / R005 / R007, O004-R002.
  - **Pluggable outputs** — R005 records *channel* of escalation;
    if escalations themselves are emitted from the product, this is
    the first product-layer demand for output channels. First time
    this pressure shows up at the product layer (per S011).
  - **Local-LLM decisioning** — R004 joins O004-R001 / R003 as
    decisioning-component pressure at architecture.
  - **Decision cache keyed by issue-data hash** — R005's per-item
    history reads onto the same per-item cache pattern as
    O004-R004.

## Deferred (still standing)

- **RSK002 "eroding credibility" intermediate clause.** Defensible
  per precedent; reviewer may try to tighten to direct close.
- **RSK001 internal fold (stale-owner + missing-chain).** Defensible
  per O001-RSK005 precedent; reviewer may try to split again.
- **R004 explicit signal list** (slip status, intervention window,
  response window). Reviewer may try "such as" framing.
- **R001 naming pressure** ("Escalation Target" vs the chain-and-
  owner content). Minor.
- **No-owner-at-all seam** between O002 and O005.
- **Channel-mismatch fold** under "right person at right time"
  reading. R005 captures channel data either way.
- **Single-mitigation fragility** on RSK003 (R004) and RSK004
  (R005). Parallel pattern.
- **R007 cross-outcome duplication** (Source Registration in O001
  and O002, identical wording). Still standing from S008. O005
  doesn't reuse.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from S004. Naming clash
  with this session's R005; unrelated content.
- **O001-RSK002 single-mitigation fragility** (R001 only). Still
  standing from S006.
- **Engineering concerns** as sharpened above — pluggable inputs,
  pluggable outputs (first surfaced S011), local-LLM decisioning,
  decision cache.
- **Vim swap files** — cleared per author in S011. No longer
  standing.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR (no doc changes; this session digest is the only
   artifact) so an independent reviewer can run session 013
   (review) against the same Outcome / Risk / Requirement rubrics.
   Reviewer-fruitful probes:
   - RSK002 "eroding credibility" intermediate — keep or tighten?
   - RSK001 internal fold — split one more time?
   - R004 explicit signal list — tighten with "such as" framing?
   - R001 naming ("Escalation Target" vs chain-and-owner content)?
   - No-owner-at-all seam between O002 and O005.
   - Single-mitigation fragilities on RSK003, RSK004.
2. After review on O005, **all five outcomes drafted, judged,
   reviewed**. J001 outcome layer complete. Next move is either
   (a) engineering layer — architecture + components, or (b) a
   product-root PDR (or set of PDRs) to capture the cross-outcome
   decisions surfaced through the outcome work: pluggable inputs,
   pluggable outputs (first surfaced S011), decisioning component,
   decision cache.
