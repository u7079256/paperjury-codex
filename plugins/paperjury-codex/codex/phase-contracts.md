# Codex Phase Contracts

This file is the Codex-native contract for PaperJury semantic phases. Use it
when Codex runs, ports, audits, or repairs the engine. Codex does not need
Claude Workflow files at runtime to know phase inputs, outputs, isolation, and
post-processing rules.

All paths in this file are repository-root relative.

## Table of Contents

- [Common Contract](#common-contract)
- [Phase Order](#phase-order)
- [assign-reviewers](#assign-reviewers)
- [reading-check](#reading-check)
- [coverage-auditor](#coverage-auditor)
- [merge](#merge)
- [trial](#trial)
- [polish](#polish)
- [recall-audit](#recall-audit)
- [drafter](#drafter)
- [edit-audit](#edit-audit)
- [meaning-audit](#meaning-audit)
- [clerk](#clerk)
- [review-panel Fast Path](#review-panel-fast-path)
- [Validation Matrix](#validation-matrix)

## Common Contract

Every semantic phase is called by the orchestrator after deterministic inputs
have been prepared. The phase receives one JSON object inline and returns strict
JSON only. No Markdown fences, no prose wrapper, and no hidden side effects.

Codex full runtime:

1. Use a fresh subagent for reviewer, juror, defense, skeptic, and audit roles
   when the host supports subagents and the user has authorized multi-agent work.
2. Pass only the text and structured fields that phase may judge. Reviewer and
   juror prompts must not receive project paths, ledger rows, prior-round notes,
   or peer outputs unless this file explicitly allows it.
3. Include the phase isolation line in the prompt.
4. Parse JSON in the orchestrator. If parsing or schema validation fails, retry
   once with the same isolated context plus the parse error. If the retry fails,
   queue the item with a visible failure reason.
5. Run deterministic validation after semantic output. Never silently drop rows.

Codex degraded runtime:

- If subagents are unavailable or unauthorized, the main session may perform the
  semantic step, but the final report must say: `Degraded PaperJury run:
  semantic isolation is prompt-level only; no parallel fresh-eye subagents were
  used.`
- A degraded run is not equivalent to the full courtroom engine.

Allowed scalar enums:

- `significance`: `major` or `minor`
- `kind`: `mechanical` or `substantive`
- trial `verdict`: `invalid-drop`, `valid-fixable`, `author-required`, or
  `escalate`
- edit audit `verdict`: `holds` or `drift`
- meaning audit `verdict`: `holds`, `weakened`, `contradicted`, or
  `now-unsupported`

## Phase Order

```text
decompose [det]
-> assign-reviewers
-> reading-check
-> coverage-auditor
-> merge
-> trial || polish
-> recall-audit
-> author gate
-> drafter
-> anchor-diff/cross-ref [det]
-> edit-audit || meaning-audit
-> apply-patch/compile-guard [det]
-> clerk
```

`[det]` phases are Node guards in `scripts/`. All other phases in this file are
semantic Codex contracts.

## assign-reviewers

Purpose: choose 2-4 holistic domain reviewers for the manuscript, default 3.

Isolation: the assignment worker may read only the manuscript text, venue
profile, persona core, and optional pinned reviewer config supplied inline.

Input:

```json
{
  "paper": "full manuscript text",
  "N": 3,
  "venueProfile": {},
  "personaCore": "shared gatekeeper prompt",
  "configReviewers": null,
  "verify": true
}
```

Output:

```json
{
  "reviewers": [
    {
      "reviewer_id": "R1",
      "domain": "specific subfield lens",
      "persona_prompt": "complete reviewer prompt"
    }
  ],
  "assignment_unverified": [],
  "subfields": ["subfield names"]
}
```

Rules:

- If `configReviewers` is supplied, preserve those slots and mark only
  unverifiable generated slots in `assignment_unverified`.
- Domains must be paper-specific, not the fixed theory/empirical/applied
  fallback unless assignment fails.
- Show the assignment to the author before review unless the project config pins
  it.

## reading-check

Purpose: each reviewer reads the whole manuscript once and reports weaknesses
plus coverage evidence.

Isolation: judge only the quoted manuscript text supplied inline. Do not read
files, do not inspect the ledger, do not use prior rounds, and do not see peer
reports.

Input:

```json
{
  "paper": "full manuscript text",
  "reviewers": [
    { "reviewer_id": "R1", "domain": "area", "persona_prompt": "..." }
  ],
  "sections": [
    { "section_path": "1", "section_title": "Introduction" }
  ],
  "venueProfile": {},
  "mode": "full",
  "targets": null
}
```

Output: one object per reviewer.

```json
{
  "reviewer_id": "R1",
  "overall_confidence": 0.82,
  "weaknesses": [
    {
      "summary": "issue summary",
      "evidence_anchor": "verbatim manuscript quote",
      "section": "Introduction",
      "significance": "major",
      "kind": "substantive",
      "references": []
    }
  ],
  "per_section_coverage": [
    {
      "section": "Introduction",
      "status": "covered",
      "in_section_quote": "verbatim quote from the section"
    }
  ]
}
```

Rules:

- `evidence_anchor` and every `in_section_quote` must be exact substrings of the
  inline manuscript.
- A reviewer that cannot quote a section did not read that section.
- Targeted re-invocation may pass only flagged sections and the same reviewer
  persona; it must not pass peer reports.

## coverage-auditor

Purpose: detect skimmed reviewer/section pairs from coverage reports.

Isolation: inspect only section names and coverage reports. Do not read the
ledger or prior rounds.

Input:

```json
{
  "paper": "full manuscript text",
  "sections": [{ "section_path": "1", "section_title": "Introduction" }],
  "reports": [
    {
      "reviewer_id": "R1",
      "per_section_coverage": [
        { "section": "Introduction", "status": "covered", "in_section_quote": "..." }
      ]
    }
  ]
}
```

Output:

```json
{
  "flags": [
    { "reviewer_id": "R1", "section": "Method", "reason": "missing quote" }
  ]
}
```

Rules:

- The orchestrator validates each flag against actual missing, invalid, or weak
  coverage evidence.
- Reinvoke flagged reviewer/section pairs at most once before routing the item
  to a visible queue.

## merge

Purpose: semantically deduplicate flattened weaknesses across reviewers.

Isolation: inspect only the flattened weakness list supplied inline. Do not
invent new issues.

Input:

```json
{
  "weaknesses": [
    {
      "idx": 0,
      "reviewer_id": "R1",
      "overall_confidence": 0.82,
      "summary": "issue",
      "evidence_anchor": "quote",
      "section": "Introduction",
      "significance": "major",
      "kind": "substantive",
      "references": []
    }
  ]
}
```

Output:

```json
{
  "issues": [
    {
      "summary": "merged issue",
      "evidence_anchor": "best quote",
      "section": "Introduction",
      "significance": "major",
      "kind": "substantive",
      "references": [],
      "raised_by": ["R1"],
      "raised_by_count": 1,
      "reviewer_confidence": 0.82,
      "close_criterion": null
    }
  ],
  "clustered": [[0]],
  "singletons": [0]
}
```

Rules:

- The semantic worker clusters indices. The orchestrator derives
  `significance` by max severity, `kind` by substantive-dominates,
  `raised_by`, corroboration, and confidence where possible.
- Every input index must appear in a cluster or become a singleton. Dropped
  indices are backfilled as singleton issues.
- Intake ledger rows start as `raised` with `close_criterion:null`.

## trial

Purpose: adjudicate substantive major charges through defense, jurors, quorum,
and judge routing.

Isolation:

- Defense may inspect the whole paper supplied inline.
- Jurors receive only the charge, local context units, optional orchestrator
  expansion text, and defense summary. They do not see peer votes, ledger rows,
  or prior rounds.
- Judge receives charge, defense, votes, and deterministic tally only.

Input:

```json
{
  "charges": [
    {
      "charge_id": "I-1",
      "section": "Introduction",
      "summary": "issue",
      "evidence_anchor": "quote",
      "significance": "major",
      "kind": "substantive",
      "references": []
    }
  ],
  "paper": "full manuscript text",
  "units": [{ "passage_id": "p1", "section": "Introduction", "text": "..." }],
  "claim_spine": [],
  "spine": [],
  "jurySize": 5,
  "escalated": false,
  "expansionsCap": 2
}
```

Output: one object per charge.

```json
{
  "charge_id": "I-1",
  "significance": "major",
  "section": "Introduction",
  "summary": "issue",
  "evidence_anchor": "quote",
  "verdict": "valid-fixable",
  "close_criterion": "One sentence must clarify the limitation.",
  "rationale": "why the verdict follows",
  "tally": { "valid": 4, "invalid": 1, "context_limited": 0 },
  "jury_size": 5,
  "escalated": false,
  "defense": "defense summary",
  "votes": [
    { "juror_id": "J1", "vote": "valid", "rationale": "..." }
  ]
}
```

Rules:

- Decide only if surviving votes reach `ceil(0.8 * jurySize)` and one side has
  more than 60 percent of surviving votes.
- Undecided tier 1 escalates to jury size 12. Undecided tier 12 becomes
  `author-required`.
- The judge sets `close_criterion` only for `valid-fixable`, and it must be
  satisfiable by editing existing text without inventing experiments or results.

## polish

Purpose: handle mechanical and minor-substantive items without full trial.

Isolation: inspect only the item batch, paper text, and venue profile supplied
inline.

Input:

```json
{
  "items": [
    {
      "issue_id": "I-2",
      "section": "Method",
      "summary": "awkward wording",
      "evidence_anchor": "quote",
      "significance": "minor",
      "kind": "mechanical"
    }
  ],
  "paper": "full manuscript text",
  "venueProfile": {}
}
```

Output:

```json
{
  "patches": [
    {
      "issue_id": "I-2",
      "kind": "mechanical",
      "before": "exact text",
      "after": "replacement text",
      "rationale": "why it is safe",
      "before_in_text": true,
      "no_op": false
    }
  ],
  "dropped": [],
  "escalate_to_trial": [],
  "flagged": []
}
```

Rules:

- Mechanical edits may draft exact-string patches.
- Minor-substantive items may be patched, dropped with a visible reason,
  escalated to trial, or flagged for author review.
- Never silently drop a flagged or suspicious item.

## recall-audit

Purpose: revive questionable drops and spot-check strong-consensus major issues
before drafting.

Isolation: inspect only supplied drops, consensus majors, units, and optional
skeptic prompts. Do not inspect ledger history except fields included inline.

Input:

```json
{
  "drops": [],
  "consensus_majors": [],
  "units": [{ "passage_id": "p1", "section": "Introduction", "text": "..." }],
  "paper": "full manuscript text",
  "skeptics": []
}
```

Output:

```json
{
  "confirmed_drops": ["I-3"],
  "revived": [
    { "charge_id": "I-4", "reason": "drop overlooked quote", "recommend": "trial" }
  ],
  "spotcheck": [
    { "charge_id": "I-5", "action": "keep", "reason": "still supported" }
  ]
}
```

Rules:

- Bias toward reviving when the drop is plausibly wrong.
- Revived charges return to trial or author-required routing; they are not
  patched directly.

## drafter

Purpose: draft minimal exact-string patches for authorized valid-fixable rows.

Isolation: inspect only authorized fixable rows, relevant units, and spine
anchors supplied inline. Do not add new experiments, numbers, citations, or
claims.

Input:

```json
{
  "venueProfile": {},
  "fixable": [
    {
      "charge_id": "I-1",
      "section": "Introduction",
      "close_criterion": "Clarify limitation.",
      "evidence_anchor": "quote"
    }
  ],
  "units": [{ "passage_id": "p1", "section": "Introduction", "text": "..." }],
  "spine": []
}
```

Output:

```json
[
  {
    "charge_id": "I-1",
    "issue_id": "I-1",
    "before": "exact existing text",
    "after": "replacement text",
    "rationale": "how it satisfies close_criterion",
    "touches_anchor": false,
    "before_in_text": true,
    "no_op": false
  }
]
```

Rules:

- `before` must be an exact substring of the current manuscript.
- Prefer the smallest edit that satisfies `close_criterion`.
- If no safe exact patch exists, return `no_op:true` with rationale instead of
  inventing text.

## edit-audit

Purpose: audit risky non-anchor edits for local sense and cross-section drift.

Isolation: inspect only the proposed edits, cross-reference hits, and supplied
passages.

Input:

```json
{
  "edits": [
    {
      "issue_id": "I-1",
      "before": "old",
      "after": "new",
      "cross_ref_hits": [
        { "token": "method name", "passage_id": "p3" }
      ]
    }
  ],
  "passages": [{ "passage_id": "p3", "text": "..." }]
}
```

Output:

```json
{
  "edit_verdicts": [
    {
      "issue_id": "I-1",
      "verdict": "holds",
      "reason": "no cross-section drift",
      "offending_text": ""
    }
  ]
}
```

Rules:

- `drift` blocks application and sends the edit to the queue.
- Do not repair the edit in this phase; only audit it.

## meaning-audit

Purpose: audit frozen anchors and argument arc after a proposed edit touches or
weakens anchor support.

Isolation: inspect only supplied anchors, baseline support, current support, and
spine entries.

Input:

```json
{
  "anchors": [
    {
      "anchor_id": "A1",
      "type": "claim",
      "frozen_text": "original claim",
      "baseline_support": "old support",
      "current_support": "new support",
      "present_verbatim": true
    }
  ],
  "spine": [
    { "anchor_id": "A1", "type": "claim", "text": "original claim" }
  ]
}
```

Output:

```json
{
  "verdicts": [
    {
      "anchor_id": "A1",
      "verdict": "holds",
      "reason": "support remains consistent",
      "offending_text": ""
    }
  ],
  "arc": { "arc_intact": true, "reason": "argument order still holds" }
}
```

Rules:

- Any `weakened`, `contradicted`, or `now-unsupported` verdict blocks
  application and queues the edit.
- Do not rewrite anchors in this phase.

## clerk

Purpose: reconcile carried issues, this round's findings, and applied edits at
the round boundary.

Isolation: inspect only carried ledger rows, this-round rows, applied edits, the
paper text, and similarity threshold supplied inline.

Input:

```json
{
  "carried": [
    {
      "ledger_id": "I-1",
      "passage_id": "p1",
      "section": "Introduction",
      "summary": "issue",
      "evidence_anchor": "quote",
      "status": "raised"
    }
  ],
  "thisRound": [],
  "appliedEdits": [
    { "issue_id": "I-1", "before": "old", "after": "new" }
  ],
  "paper": "current manuscript text",
  "simThreshold": 0.82
}
```

Output:

```json
{
  "reconciled": [
    { "ledger_id": "I-1", "outcome": "closed", "reason": "criterion satisfied" }
  ],
  "merges": [
    { "this_round_id": "I-9", "into": "I-1" }
  ],
  "genuinely_new": ["I-10"],
  "genuinely_new_count": 1,
  "new_closures_count": 1,
  "new_author_required_count": 0,
  "converged": false
}
```

Rules:

- The orchestrator applies ledger state changes; the clerk only recommends.
- Merge keys use `passage_id` plus semantic similarity, not summary text alone.
- A round can converge only after clerk reconciliation and deterministic ledger
  gates have passed.

## review-panel Fast Path

Purpose: quick single-pass review when the user asks for a lightweight check or
the full courtroom engine is too expensive.

Isolation: same as `reading-check`; panel reviewers are isolated from one
another and receive no ledger or prior-round state.

Contract:

- Run three broad lenses or configured reviewers.
- Collect weaknesses with exact evidence quotes.
- Merge by the same no-silent-drop rule as `merge`.
- Label output as a quick panel, not a full courtroom round.

The fast path may populate the ledger only when the user explicitly wants the
findings carried into the review-revise loop.

## Validation Matrix

The orchestrator must enforce:

| Phase | Deterministic validation |
|---|---|
| `assign-reviewers` | reviewer count 2-4 unless config overrides; every reviewer has id, domain, prompt |
| `reading-check` | every evidence quote is an exact substring; every section has coverage or a flag |
| `coverage-auditor` | flags correspond to missing, invalid, or weak coverage |
| `merge` | every weakness index is clustered or singleton-backfilled |
| `trial` | quorum and majority math recomputed outside the model; close criterion only for valid-fixable |
| `polish` | every item appears in patches, dropped, escalate_to_trial, or flagged |
| `recall-audit` | revived rows re-enter routing; drops are visible |
| `drafter` | `before` exact substring; no-op queued instead of fabricated patch |
| `edit-audit` | any drift blocks application |
| `meaning-audit` | any weakened/contradicted/unsupported anchor blocks application |
| `clerk` | ledger mutations applied only through the orchestrator; gate checked after reconciliation |
