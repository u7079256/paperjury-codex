# PaperJury for Codex

Audience: a Codex agent or user installing or running PaperJury in a Codex
environment.

All paths in this file are repository-root relative.

## One-paragraph model

PaperJury is a pre-submission CS-paper editing and adversarial review skill. In
Codex, `SKILL.md` provides the trigger and operating manual, `scripts/*.js` run
deterministic checks, `.paper-review/LEDGER.json` stores state, and
`codex/runtime.md` defines Codex orchestration. `codex/phase-contracts.md`
defines Codex-native phase inputs, outputs, isolation, and validation rules.
Claude Workflow files are not required in this repository.

## Install

User-level Codex skill install:

```powershell
git clone https://github.com/u7079256/paperjury-codex "$env:USERPROFILE\.codex\skills\paperjury"
```

Requirements:

- `node` for deterministic guards.
- `git` for ordinary project work and diffing.
- Optional `latexmk` or `pdflatex`; without them, compile checks degrade to
  structural lint and report `compiled:null`.

Verify the skill folder:

```powershell
python <codex-skill-creator>\scripts\quick_validate.py <path-to-paperjury>
node <path-to-paperjury>\scripts\doctor.js --project <paper-project>
```

## Modes

| Mode | Trigger | Codex behavior |
|---|---|---|
| `direct-edit` | One concrete LaTeX edit: polish, de-AI, translate a note, compress, rewrite caption | Locate passage, draft patch, ask for author sign-off, apply, run narrow checks |
| `review` | Review, critique, mock-review, pre-submission hardening | Run one courtroom round, using Codex subagents when explicitly authorized |
| `auto` | Explicit Codex goal or `/goal`-style request | Run the review-revise loop under a verifiable ledger gate after up-front author sign-off |

Auto is never self-detected. A normal review prompt does not authorize an
unattended loop.

## Codex-specific execution rule

Before review or auto fan-out, read `codex/runtime.md` and
`codex/phase-contracts.md`. The key rule: Codex subagents are used
only when the user has explicitly authorized multi-agent work. Without that, run
a labeled degraded path or ask for authorization.

## Engine summary

The v3 engine remains:

```text
decompose -> assign-reviewers -> reading-check -> coverage-auditor -> merge
-> trial || polish -> recall-audit -> drafter -> edit-audit || meaning-audit
-> apply/compile guards -> clerk
```

The state gate is deterministic:

```powershell
node scripts/ledger.js gate <ledger.json>
node scripts/ledger.js unadjudicated <ledger.json>
```

The gate passes only when there are zero gate-blocking active majors and no
unadjudicated active major rows.

## File map

| Need | File |
|---|---|
| Codex runtime mapping | `codex/runtime.md` |
| Codex phase contracts | `codex/phase-contracts.md` |
| Engine contracts | `references/review-engine-v3.md` |
| Ledger schema | `references/ledger-schema.md` |
| Auto checklist | `references/auto-mode.md` |
| Reviewer personas | `references/reviewer-personas.md` |
| Writing toolkit | `references/writing-toolkit.md` |
| Deterministic guards | `scripts/` |
| Historical Claude source | original `paperjury` repository |

When documents conflict, prefer this order for Codex: `codex/runtime.md`,
then `codex/phase-contracts.md`, then `references/review-engine-v3.md`,
then `SKILL.md`, then README files.
