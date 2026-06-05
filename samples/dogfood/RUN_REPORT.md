# PaperJury Revision Comparison

**Input (draft):** `original_draft.pdf` (origin draft)
**Process:** one Codex-adapted AUTO-mode PaperJury review round
**Output:** `revised_draft.pdf` (21 pp, compiles with 0 errors, 16 overfull hbox warnings)

Ledger: 12 reviewer issues -> 5 applied / 1 queued / 6 author-required.

## Table 1 — Fixable defects F1-F6

| Problem (location) | Fix result | Human-verified |
|---|---|---|
| **F1** section 8 concurrency: prose says `8`, table says `16` (self-contradiction) | Unified to `16` in the revised draft | verified, self-contradiction resolved |
| **F2** section 7.2 clerk merge threshold: prose `simThreshold=0.7` vs adjacent equation `0.8` | **Not fixed: queued by bounded-aggressive policy; revised draft still says `0.7`** | pending author / policy override (prose still conflicts with equation) |
| **F3** section 5 escalation jury: written `jurySize=10`, elsewhere `12` | Changed to `12` | verified, inconsistency resolved |
| **F4** section 2 isolation invariant flipped ("reviewers are given the cumulative ledger") | **Not fixed: the flipped isolation sentence is still in the revised draft** | pending author / edit pass |
| **F5** section 1 C5 term written `registrar`, called `clerk` elsewhere | **Not fixed: `registrar` is still in the revised draft** | pending author / edit pass |
| **F6** section 4 assignment paragraph carried an unnecessary external marker | Removed the marker; sentence kept | verified, build check passes |

## Table 2 — Fabricated claims A1-A3 (unsupported assertions)

| Problem (location) | Fix result | Human-verified |
|---|---|---|
| **A1** abstract asserts "94% router agreement ... confirming ... in practice" (no experiment) | **Not fixed: routed author-required; assertion is still in the revised draft** | pending author (soften or delete) |
| **A2** section 4 asserts "in our runs ... order of magnitude fewer agents ... strictly higher precision" (no experiment) | **Not fixed: routed author-required; assertion is still in the revised draft** | pending author (soften or delete) |
| **A3** section 3 asserts "reaches this fixed point within three rounds" (no data) | **Not fixed: missed by the Codex ledger; sentence is still in the revised draft** | pending author / review pass |

## Table 3 — Baits B1-B2 (look like flaws, are defensible; must stay untouched)

| Problem (location) | Fix result | Human-verified |
|---|---|---|
| **B1** section 4 "no per-section reviewer assignment and no per-section coverage quota" (looks like a coverage gap) | No fix needed; clause kept verbatim | verified (correctly left untouched, no false positive) |
| **B2** section 7 gate "evaluated over the same ledger state the engine's own steps write" (looks circular) | No fix needed; clause kept verbatim | verified (correctly left untouched, no false positive) |

## Summary
- Fixable defects **F1, F3, and F6 are resolved**; F2/F4/F5 remain pending.
- **6 items pending author or another edit pass:** F2, F4, F5, A1, A2, and A3.
- Baits B1/B2 were kept verbatim with zero false positives.
- Output `revised_draft.pdf` compiles to 21 pp with 0 errors; 16 overfull hbox warnings remain.
