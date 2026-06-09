# Session 002 — Judged product root

## Goal

Run the judge pass on `docs/product/README.md` (drafted in session 001)
against the Product / Job / Outcome rubrics in the `intent` skill,
before moving downstream to outcomes.

## Context loaded

- `docs/product/README.md` — the draft under judgment.
- `resources/previous-sessions/001-product-root-drafted.md` — what
  session 001 decided and deferred.
- `resources/bird-dogging/bird-dogging-overview.md` — the practice the
  product serves.
- `intent` skill: `SKILL.md` (Product / Job / Outcome / Records),
  `references/workflows.md`, `references/context-tracing.md`,
  `assets/templates/product.md`, `assets/templates/outcome.md`.

## Workspace path mismatch (flagged)

Cursor session opened against `/Users/max.dunn/dev/personal/jomadu/
bird-dog-skill`, which does not exist on disk. Actual repo is
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogging-skill`
(remote `github.com/colchuck-ai/bird-dogging-skill`). Worked from the
real repo. Worth resolving so future sessions don't get pointed at the
wrong path.

## Workflow status caught early

Session 001 was committed straight to `main` — no judge or review pass.
That meant session 001's "recommended next step" (jump to drafting
O003) would have inherited unchecked product framing. Surfaced the
tradeoff; user chose judge-first.

## Judge findings

Material (fix before going downstream):

1. **Name drift.** H1 "Bird Dogging", session 001 notes "Bird Dog", repo
   `bird-dogging-skill`. Tree should read as one story.
2. **Job ↔ outcomes coverage gap.** Job promised "lands on time" but no
   outcome measured lateness / commitment-hit-rate. Either soften the
   job or add the outcome.

Smaller (do now):

3. Lead paragraph mixed persona, job, and means. "Efficient and
   reliable" was outcome-flavored (and overlapped the dropped Cycle
   Effort).
4. Persona "Tech PMs and others" too vague vs. session 001's locked
   "Tech PM / TPM driving cross-team work".

Flagged, deferred:

5. **O002 vs O001 boundary** needs one clarifying sentence (an item
   forgotten between checkpoints also reads as detection lag). Belongs
   in O002's own doc.
6. **O006 Stakeholder Update Speed.** J001 doesn't mention stakeholders;
   reporting may be a sibling job, not part of J001.
7. **O005 Escalation Quality** packs targeting + timing into one
   outcome. Acceptable now; expect it to split into wrong-person /
   wrong-time risks when O005's outcome doc is drafted.

## Locked decisions

- **Name:** Bird Dogging (not Bird Dog).
- **Fix 2 path:** soften the job, don't add a new outcome. Lower-risk
  and symmetric with session 001's deferral of Cycle Effort. If
  lateness matters, surface it later as a risk under O001/O002 or as a
  new outcome.
- **O006 dropped from J001** as out-of-scope. May become a separate
  job (e.g. J002 — drive stakeholder visibility) later; not added now.

## What got edited

`docs/product/README.md`:

- Lead paragraph rewritten to: persona (Tech PMs / TPMs driving
  cross-team work they don't directly control) + scope (supports the
  chase loop — driving open items across teams to completion) + benefit
  (commitments land without the bird-dogger doing the work themselves).
  Dropped "efficient and reliable" and "and others".
- Job J001 statement: replaced "I want to make sure each open item
  lands on time" with "I want each open item to get to closure without
  falling through".
- Removed O006. Final outcomes: O001 Slip Detection Lag, O002 Coverage,
  O003 Status Fidelity, O004 Triage Speed, O005 Escalation Quality. No
  renumbering — O006's slot stays burned.

No Change Record. Create + judge + review form one cycle; revisions
during judge are not a CR.

Changes uncommitted at session end. Open question for the author:
branch + PR these edits (actually run session 3 / review), or merge to
`main` again and skip review.

## Deferred (still standing from session 001)

- Risks under O003 (Status Fidelity): LLM mis-judges status; stale
  decision cache after tracker schema change; tracker data itself
  wrong or lagging.
- Risks under O001 / O002: tracker outage hides slip; new tracker
  source not integrated.
- Engineering concerns: pluggable tracker inputs, pluggable cache,
  pluggable outputs, local-LLM decisioning, decision cache keyed by
  issue-data hash.

## Recommended next step

1. Decide branch + PR vs. direct-to-`main` for these judge-pass edits.
   Recommend branch + PR so session 3 actually happens.
2. Once the product root is settled, draft
   `docs/product/outcomes/J001-O003-status-fidelity/README.md` first
   (session 001's recommendation; risks on O003 drive most of the
   deferred engineering pluggability).
