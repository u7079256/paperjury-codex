# Codex runtime adapter

This file is the authoritative Codex runtime guide for PaperJury. The engine
design in `references/review-engine-v3.md` is still the semantic source of
truth, and `codex/phase-contracts.md` is the Codex-native contract for semantic
phase inputs, outputs, isolation, and validation.

All paths in this file are repository-root relative.

## Runtime Components

| PaperJury primitive | Codex runtime |
|---|---|
| Skill trigger | `SKILL.md` in `.codex/skills` or project skill roots |
| Semantic fan-out | Codex subagents when explicitly authorized; otherwise degraded main-session execution |
| Deterministic guards | Orchestrator-side `node scripts/*.js` |
| Durable memory | Ledger plus project-owned `.paper-review/CONVENTIONS.md` or host memory |
| Auto driver | Codex active goal or explicit user goal request |
| Structured output | Prompted strict JSON plus orchestrator validation |

Use `codex/phase-contracts.md` as the runtime phase contract. This repository
does not require Claude Workflow files for operation.

## Installation for Codex

Place the repository where Codex discovers skills:

```powershell
git clone https://github.com/u7079256/paperjury-codex "$env:USERPROFILE\.codex\skills\paperjury"
```

Project-scoped installation is also valid when a Codex environment supports a
project skill root. `node` is required. `git` is recommended. A LaTeX toolchain is
optional because `compile-guard.js` reports `compiled:null` when it cannot compile.

## Runtime Preconditions

Resolve all inputs at runtime:

- `manuscript`: the named `.tex` file, or the `.tex` containing both
  `\documentclass` and `\begin{document}`. Ask if several candidates exist.
- `venue_family`: user-provided, config-pinned, or inferred from style/class
  context. Ask if unclear.
- `ledger`: default to `<manuscript-dir>/.paper-review/LEDGER.json`.
- `journal`: default to `<manuscript-dir>/.paper-review/journal.jsonl`.
- `spine`: default to `<manuscript-dir>/.paper-review/spine.json`.
- `author`: the person who can authorize edits.

All state belongs to the paper project, not the skill repository.

## Subagent Authorization

Codex hosts may expose subagents behind an explicit user-authorization rule.
PaperJury review and auto modes need multi-agent fan-out to preserve the design.

Before using Codex subagents, ensure the user has explicitly asked for, or clearly
authorized, parallel agents/subagents for PaperJury. If not, ask a targeted
question before the first fan-out:

```text
PaperJury review mode needs isolated reviewer and jury subagents. Do you authorize Codex to use parallel subagents for this run?
```

If the answer is no or the host has no subagent tool, continue only in degraded
mode and label the result:

```text
Degraded PaperJury run: semantic isolation is prompt-level only; no parallel fresh-eye subagents were used.
```

Do not mark a degraded run as equivalent to the full courtroom engine.

## Codex Subagent Pattern

For each semantic fan-out job:

1. Spawn a fresh subagent with no inherited thread context when the host supports
   it, for example `fork_context:false`.
2. Inline the only context it may judge. Do not pass file paths unless the phase
   explicitly needs a file-reading worker, which reviewer and juror phases do not.
3. Include the phase isolation line from `codex/phase-contracts.md`.
4. Demand strict JSON matching the phase contract in
   `codex/phase-contracts.md`.
5. Parse and validate the JSON in the orchestrator. If invalid, retry once with
   the same isolated prompt plus the parse error. If still invalid, queue the
   item with a reason instead of silently dropping it.
6. Close or stop subagents after collecting their final output.

The orchestrator may use the main session for one-agent semantic steps such as
`merge` only when doing so does not leak peer outputs into a later reviewer or
juror. Reviewer and juror roles should be subagents in full mode.

## Deterministic Guard Pattern

Run guards from the skill directory with Node:

```powershell
node scripts/decompose.js passages <main.tex>
node scripts/decompose.js units <main.tex>
node scripts/ledger.js init <ledger.json> --manuscript <main.tex> --venue <vision|nlp|ml> --round 1
node scripts/ledger.js gate <ledger.json>
node scripts/ledger.js unadjudicated <ledger.json>
node scripts/compile-guard.js check <main.tex>
```

Mutating guards write only project-owned state or the authorized manuscript edit.
`apply-patch.js` may be called only after the relevant author-signoff gate.

## Review Mode: One Codex Round

`[det]` means local Node script. `[SF]` means semantic fan-out under the contract
in `codex/phase-contracts.md`: Codex subagents when authorized,
degraded main-session execution otherwise. `[LEDGER]` means a write through
`scripts/ledger.js` or its module contract.

1. Resolve inputs and read any project config or conventions.
2. `[det]` Run `decompose.js passages` and `decompose.js units`.
3. `[SF]` `assign-reviewers`: create 2-4 domain reviewers, default 3. Show the
   assignment to the author before review unless it is config-pinned.
4. `[SF]` `reading-check`: spawn one isolated reviewer per reviewer profile. Each
   receives the whole manuscript text inline, the canonical section list, the
   venue profile, and strict JSON output requirements.
5. `[det]` Quote-verify coverage and weakness evidence against the inline text.
6. `[SF]` `coverage-auditor`: detect skimmed reviewer/section pairs. Reinvoke only
   the flagged reviewer/section pairs once.
7. `[det]` Flatten reviewer weaknesses with `reviewer_id` and
   `reviewer_confidence`.
8. `[SF]` `merge`: semantically cluster weaknesses. Backfill every unclustered
   input weakness as a singleton. `[LEDGER]` Add issues as `raised` with
   `close_criterion:null` and `passage_id` from decompose.
9. `[det]` Route rows: mechanical and minor-substantive to `polish`;
   substantive-major to `trial`.
10. `[SF]` `trial`: for each substantive-major charge, run whole-paper defense,
    decorrelated jurors, deterministic quorum/majority, and judge routing. Escalate
    undecided tier-1 charges to tier 12. `[LEDGER]` Store verdicts, tallies, and
    `close_criterion` for `valid-fixable`.
11. `[SF]` `polish`: draft or queue mechanical and minor-substantive items. Never
    silently drop a flagged item.
12. `[SF]` `recall-audit`: revive questionable drops and spot-check strong-consensus
    majors before drafting.
13. `[HUMAN]` In review mode, ask the author before applying any patch.
14. `[SF]` `drafter`: draft exact-string patches for authorized valid-fixable rows.
15. `[det]` Run edit-safety prefilters: `anchor-diff.js` and `cross-ref.js`.
16. `[SF]` For risky edits, run `meaning-audit` for frozen anchors or `edit-audit`
    for risky non-anchor edits.
17. `[det]` Apply exact patches with `apply-patch.js`, run `compile-guard.js`, and
    revert/queue on failure. `[LEDGER]` Mark verified rows `closed`.
18. `[SF]` `clerk`: reconcile re-raises and carried queue items. `[LEDGER]` Apply
    merges and terminal states.
19. Report counts, gate status, queued/author-required items, and whether this was
    full or degraded runtime.

Review mode stops after the round or at a human gate. It does not automatically
start the next round.

## Auto Mode in Codex

Auto is explicit only. Do not infer it from tool permissions or from a normal
review request.

Codex auto requires:

1. User gives or resumes a goal whose completion can be checked by
   `node scripts/ledger.js gate <ledger.json>` and
   `node scripts/ledger.js unadjudicated <ledger.json>`.
2. Author confirms the frozen spine and reviewer assignment before the unattended
   loop.
3. Author confirms the bounded-aggressive policy: apply only low-risk,
   non-anchor, compile-passing patches that satisfy a `close_criterion`; queue
   everything else.

The orchestrator must not create a Codex goal unless the user explicitly asks for
one. If a goal is active, completion is proven only when:

- `ledger.js gate` exits PASS,
- `ledger.js unadjudicated` is empty,
- the current round has completed its clerk reconciliation,
- any applied patches have a journal entry and compile/lint result,
- queued and author-required rows are reported, not hidden.

## Isolation Limits

Codex prompt isolation is not the same as a hard filesystem sandbox. A Codex
subagent may still have access to the project workspace depending on host policy.
PaperJury mitigates this by:

- passing the manuscript text inline,
- passing no project file paths to reviewer/juror prompts,
- including explicit no-file-read isolation instructions,
- validating quotes against the inline manuscript,
- reporting degraded status when hard isolation cannot be guaranteed.

Do not state that Codex has hard read-whitelist isolation unless the host actually
enforces it.

## How to Use Phase Contracts in Codex

For each semantic phase:

- Read `codex/phase-contracts.md`.
- Use that phase's input object, isolation rule, output shape, and deterministic
  validation matrix.
- Preserve the same output shape so `review-engine-v3.md` seams remain valid.
- Do not rely on host-specific DSL globals. The phase contract is ordinary JSON
  plus natural-language isolation rules.

## Static Validation Checklist

For a theoretical migration check that does not run a real paper scenario:

```powershell
node scripts/doctor.js --project <paper-project>
python <skill-creator>/scripts/quick_validate.py <paperjury-dir>
git status --short
```

Also inspect that:

- `SKILL.md` frontmatter has Codex-allowed keys.
- `codex/phase-contracts.md` covers every semantic phase and requires
  strict JSON output.
- Local documentation links resolve.
- Node scripts parse.
- No README or guide claims that Codex requires Claude Workflow files.
- Config examples point the ledger at `LEDGER.json`, not only `LEDGER.md`.
