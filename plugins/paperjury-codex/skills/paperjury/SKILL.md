---
name: paperjury
description: Pre-submission CS-conference LaTeX paper editing and adversarial review. Use for direct edits such as polish, de-AI, translating Chinese notes to LaTeX, compressing passages, captions, and experiment prose; for review or critique requests such as review, mock-review, 审稿, 评审; and for explicit auto or goal-driven review-revise loops.
---

# PaperJury Plugin Entry

This skill is packaged by the `paperjury-codex` Codex plugin. The canonical
PaperJury workflow lives at the plugin root in `../../SKILL.md`; read that file
first and follow it as the source of truth.

Resolve the plugin-owned support files from the plugin root, not from this
wrapper directory:

- deterministic scripts: `../../scripts/`
- review and writing references: `../../references/`
- Codex runtime notes and phase contracts: `../../codex/`
- config template: `../../configs/config-template.md`
- fallback agent metadata/personas: `../../agents/`

Project-specific paper files, ledgers, journals, and generated patches belong in
the user's active paper project. Do not write them into the plugin directory.
