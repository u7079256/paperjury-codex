# PaperJury Codex Instructions

PaperJury Codex is a Codex-first skill. Use `codex/phase-contracts.md` as the
prompt/schema/isolation contract and follow `codex/runtime.md` for runtime
orchestration.

All paths in this file are repository-root relative.

## Load Order

For any Codex task that uses, ports, audits, or changes PaperJury:

1. Read `SKILL.md`.
2. If the task involves review, auto mode, fan-out, or subagents, read
   `codex/runtime.md`.
3. For Codex semantic phase work, read `codex/phase-contracts.md`.
4. For engine changes, read `references/review-engine-v3.md` and
   `references/ledger-schema.md`.
5. For user-facing install or usage docs, also read `codex/AGENT-GUIDE.md`.

## Codex Runtime Rules

- Do not require Claude Workflow files for Codex operation.
- Use `codex/phase-contracts.md` for semantic phase inputs, outputs,
  isolation lines, and deterministic validation rules.
- Use `node scripts/*.js` for deterministic guards.
- Use Codex subagents for semantic fan-out only when the user has explicitly
  authorized multi-agent work. If subagents are unavailable or unauthorized, run
  a clearly labeled degraded path and do not claim full isolation.
- Keep reviewer and juror prompts isolated: inline only the manuscript/context
  they may judge, pass no project file paths, no ledger, no prior-round reports,
  and no peer outputs.
- Never edit a manuscript without explicit author sign-off. Auto mode requires
  up-front policy sign-off and queues anything outside the authorized envelope.
- Store mutable state in the project-owned `.paper-review/` directory:
  `LEDGER.json`, rendered `LEDGER.md`, `journal.jsonl`, `spine.json`, and any
  stable `CONVENTIONS.md`.
- Do not hardcode local paper paths into the skill.

## Validation

Before finishing a migration or runtime change, run static checks when possible:

- `node scripts/doctor.js --project <paper-project>`
- `python <codex-skill-creator>/scripts/quick_validate.py <paperjury-dir>`
- `git status --short`

Do not run a real paper-review scenario unless the user explicitly asks for it.
