[中文](README.md) · **English**

<p align="center">
  <img src="docs/paperjury-mark.png" alt="PaperJury logo" width="96">
</p>

# PaperJury

> A pre-submission AI review stress-test for research papers.

<p align="center">
  <a href="https://arxiv.org/abs/2606.16322"><img alt="Read the paper on arXiv" src="https://img.shields.io/badge/arXiv-2606.16322-b31b1b?style=for-the-badge&logo=arxiv&logoColor=white"></a>
  <a href="https://u7079256.github.io/paperjury/overview.html?lang=en"><img alt="Open the live interactive overview" src="https://img.shields.io/badge/Open_the_interactive_overview-d6a14b?style=for-the-badge&logo=githubpages&logoColor=white"></a>
  <a href="https://github.com/u7079256/paperjury"><img alt="Claude Code version" src="https://img.shields.io/badge/Claude_Code_version-2b2d42?style=for-the-badge"></a>
  <a href="https://github.com/u7079256/paperjury-codex/tree/main/samples/dogfood"><img alt="View the dogfood sample" src="https://img.shields.io/badge/Sample-Dogfood-2f7d55?style=for-the-badge"></a>
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-3b3d47?style=for-the-badge">
</p>

*Before a reviewer tears it apart, let a jury do it first.*

PaperJury turns paper feedback into a closed loop: review → verdict → revise → verify. Instead of taking every AI suggestion at face value, it sorts each issue into one of three outcomes:

- **Fixable**: safe, text-level issues that can be patched automatically.
- **Author-required**: missing experiments, missing evidence, or research decisions that stay with you.
- **Invalid**: reviewer misreadings or unsupported critiques that should not be applied.

It offers three modes: direct-edit, review, and auto. PaperJury is built for pre-submission self-checking. It does not replace peer review, it does not invent missing experiments, and it keeps research-level decisions with the author.

> **RedNote community milestone:** the PaperJury post has reached **30k views** and **1.8k saves**. Thanks for helping it reach more people writing and revising papers.

Interactive overview: the [live site](https://u7079256.github.io/paperjury/overview.html?lang=en) (GitHub Pages), or [`docs/overview.html`](docs/overview.html) in-repo.

---

## What you get

| Output | What it contains |
|---|---|
| **Issue ledger** | Evidence, location, verdict, and status for every reviewer-style issue. |
| **Reviewable patches** | Minimal edits for safe fixes only; risky edits are queued for author judgment. |
| **Verification report** | Real LaTeX and formatting checks when the toolchain exists; explicit degradation when it does not. |
| **Dogfood sample** | The repo-level [dogfood sample](https://github.com/u7079256/paperjury-codex/tree/main/samples/dogfood) includes before/after PDFs and a human-verified run report. |

---

## 📰 News

- 📄 **The PaperJury paper is on arXiv.** Read it here: [*PaperJury: Due-Process Review for Bounded LaTeX Revision*](https://arxiv.org/abs/2606.16322) (arXiv:2606.16322) — the full review → verdict → revise → verify engine written up as a paper.
- 🚀 **Codex plugin released.** PaperJury Codex is now packaged for the Codex plugin marketplace route. If you are interested in pre-submission paper review workflows, please ⭐ star this repo; we would also greatly appreciate issues, feedback, and suggestions.
- 🔔 **v1.0 release:** the Codex plugin now includes a non-blocking update reminder that points users to the latest stable release when a newer tag exists.
- 🧪 **Dogfood sample added:** this repo now includes a compact dogfood sample with before/after PDFs and a human-verified run report.

## ✅ TODO

- [x] 🔌 **Codex plugin marketplace release.** Package PaperJury for direct install through the Codex plugin marketplace route, alongside the legacy clone install.
- [x] 🔔 **Soft update reminders.** Check for newer stable release tags at PaperJury startup and show a non-blocking update notice.
- [ ] ⚡ **Fast mode / quick version.** A lower-latency, lower-token path for fast checks when you want useful triage more than full courtroom depth.
- [x] 🧩 **Claude Code plugin marketplace release.** Package the Claude Code version for direct install from the Claude Code plugin marketplace, alongside the clone install in the original PaperJury repo.

---

## Responsible Use

PaperJury is a pre-submission self-check workflow. It does not replace the author's scientific judgment, and it does not replace peer review. It should never be used to invent experiments, fabricate results, add unsupported claims, or hide a paper's limitations.

When an issue needs a new experiment, missing evidence, private knowledge, or a research-level decision, PaperJury routes it to the author instead of patching it automatically. The Fixable / Author-required / Invalid outcomes exist precisely so that judgment calls stay with you.

The intended use is to surface avoidable problems earlier, while you can still act on them: unclear claims, weak logical connections, unsupported wording, formatting risks, and the kind of reviewer-style concerns worth checking before submission.

---

## Install

PaperJury Codex is packaged as a Codex plugin. The Claude Code plugin/version
lives in the original [paperjury](https://github.com/u7079256/paperjury)
repository; this repository does not ship or execute Claude Workflow files.

```powershell
# Latest channel: use the current main branch when installing or reinstalling.
codex marketplace add u7079256/paperjury-codex

# Stable pinned release: reproducible v1.0 install.
codex marketplace add u7079256/paperjury-codex@v1.0
```

Then install **PaperJury Codex** from the Codex plugin UI. The plugin exposes the
`paperjury` skill and ships its deterministic Node guards in the plugin package.
`node` is required for deterministic checks; a LaTeX toolchain is optional
because compile checks degrade honestly when it is absent.

At the start of a PaperJury run, the plugin performs a soft update check against
stable GitHub release tags. If a newer tag exists, it prints the latest-channel
and pinned-release install commands; if GitHub is unreachable, it stays silent
and continues. Set `PAPERJURY_DISABLE_UPDATE_CHECK=1` to disable this reminder.
After reinstalling or changing release channels, start a new Codex thread so the
updated skill content is loaded.

If your Codex surface does not yet expose plugin installation, the legacy skill
install still works:

```powershell
git clone https://github.com/u7079256/paperjury-codex "$env:USERPROFILE\.codex\skills\paperjury"
```

**For Codex / coding agents:** read [`codex/AGENT-GUIDE.md`](codex/AGENT-GUIDE.md)
and [`codex/runtime.md`](codex/runtime.md), then use
[`codex/phase-contracts.md`](codex/phase-contracts.md) for
phase inputs, outputs, isolation, and validation.

---

## Why it works

Most writing tools only push your paper forward: they draft and they polish. None of them argues the other side of your claims the way a reviewer will. PaperJury is built around that gap, in four parts.

- **Adversarial by construction.** Your paper gets due process, not one pass of suggestions: N domain reviewers read the whole paper, a contestability router sends the real disputes to a two-sided trial, a jury of 5 (escalating to 12 only when it cannot reach a clear majority) deliberates under isolation, and a judge returns one of three verdicts: fix it, needs you, or no fix. A verdict can land "no fix", which a yes-and rewriter structurally cannot return.
- **Closed-loop, not forward-only.** Each round is a clean re-review of the edited paper (the panel never sees the prior ledger, so a re-raised issue is real corroboration, not anchoring), and a deterministic clerk reconciles every round into one ledger until a clean round surfaces nothing new. Before any edit, fresh skeptics try to revive whatever got wrongly dropped and stress-test strong-consensus verdicts.
- **Guardrails, not autopilot.** Safe fixes land under risk-proportional safety (frozen anchors, a per-passage edit cap, an anchor and cross-section meaning audit), always behind your sign-off. Risky edits are not applied silently; they queue for one human pass.
- **Real compile, not just critique.** It runs an actual LaTeX build on your machine and reports true errors, undefined refs, overfull boxes, and the page count, or degrades honestly to a structural lint when no toolchain is present. Deterministic desk-reject checks catch the classics: anonymization leaks, margin and spacing hacks, documentclass drift, missing required sections, and page-limit overflow, checked against your project's own constraints.

---

## Three modes

### Direct-Edit (common)

- **Trigger:** describe a change in Chinese or English and have the LaTeX edited directly.
- **Example utterances:** "把这段改成…", "polish this paragraph", "把我对 intro 的想法写成 LaTeX", "tighten this".
- **Behavior:** no review panel; go straight to drafting the patch through the writing toolkit, with author sign-off.

### Review (occasional)

- **Trigger:** ask for the paper to be critiqued or hardened: review / critique / 审稿 / 评审 / mock-review, or iterating a draft to clear reviewer-raised issues.
- **Behavior:** runs the courtroom review engine (`references/review-engine-v3.md`).
- **Scope sub-trigger:** `full` (whole paper) or `passage` (one section / paragraph / claim).

### Auto (unattended)

- **Trigger (explicit only):** opt in via `/goal` (or config `mode: auto`) to run the review-revise loop unattended toward a verifiable goal.
- **Hard constraint:** **auto is never self-detected; it is explicit only.** There is no runtime signal for it, so it is entered only via a `/goal` context or a project config `mode: auto`.
- **Behavior:** establish the `spine` and the reviewer assignment up front (the human steps), then the engine applies safe fixes under the bounded-aggressive + edit-safety policy, queues the rest, and runs multiple rounds until it stops: on clerk convergence, or an applied-quiescence / hard-limit backstop. See `references/auto-mode.md`.

---

## Usage examples: what to do when

You don't run commands; you say what you want and the skill picks the mode.

**Edit one thing (the everyday case → direct-edit):**
- "Polish this paragraph." / "把这段 intro 改紧一些。"
- "Turn my Chinese note for the intro into LaTeX: `<your idea>`."
- "De-AI this paragraph." / "Compress this sentence to one line." / "Rewrite this caption."
- → it drafts the LaTeX change, self-checks it, shows you the patch, and applies it after you approve. No panel.

**Get the paper critiqued before submission (→ review):**
- "Review my paper." / "审稿。" / "Mock-review this before I submit."
- "Critique just Section 3.2." / "review passage `<the claim you paste>`."
- "Here are the issues a reviewer raised; iterate the draft to clear them."
- → it runs the adversarial engine, surfaces the real weaknesses (separating fatal flaws from nits), and walks you through each: you give direction, it drafts fixes you authorize. Nothing changes without your sign-off.

**Harden it unattended toward a goal (→ auto, needs `/goal`):**
- `/goal "harden the paper until ledger.js gate passes (0 gate-blocking major)"`
- → it runs the review-revise loop across many rounds on its own, applying safe fixes and queueing risky ones for one pass when you return. This needs an explicit Codex goal or equivalent multi-turn driver: turning on tool permission and sending a normal prompt runs one round and stops, it does not loop (see [`codex/AGENT-GUIDE.md`](codex/AGENT-GUIDE.md)).

**Make sure it won't get desk-rejected:**
- "Run the submission-readiness / compliance check." → deterministic format screening + a compile-driven layout check.

Rule of thumb: **one change → just say it; want it picked apart → say "review"; want it run unattended → `/goal`.**

---

## Technical details

If you only want to use PaperJury, you can skip this section. If you want the mechanism, source layout, or agent-driving details, start here:

| What you want to inspect | Entry point |
|---|---|
| Real run output | [dogfood sample](https://github.com/u7079256/paperjury-codex/tree/main/samples/dogfood) |
| Codex runtime / agent driving guide | [`codex/AGENT-GUIDE.md`](codex/AGENT-GUIDE.md) · [`codex/runtime.md`](codex/runtime.md) |
| Semantic phase contracts | [`codex/phase-contracts.md`](codex/phase-contracts.md) |
| Full protocol and ledger state machine | [`references/review-engine-v3.md`](references/review-engine-v3.md) · [`references/ledger-schema.md`](references/ledger-schema.md) |
| Visual overview | [live interactive overview](https://u7079256.github.io/paperjury/overview.html?lang=en) |

<details>
<summary><b>Expand engine, runtime, and repository details</b></summary>

### Engine overview

The courtroom engine is `assign-reviewers → reading-check → coverage-auditor → merge → {trial ‖ polish} → recall-audit → drafter → {edit-audit | meaning-audit} → clerk`. Generation is bounded (N holistic domain reviewers, not a per-(unit × lens) flood); adjudication is routed by contestability; edits are guarded by risk; the multi-round loop converges via a deterministic clerk. The **deterministic guards in `scripts/`** run orchestrator-side via Node between semantic calls.

### Deterministic stages (orchestrator-side, Node via Bash)

1. `decompose`: split manuscript into reading units, the canonical section list, and stable `passage-id`s (which prevent text drift and give jurors local context).
2. `spine` (auto only): extract anchors, author confirm, freeze → `spine.json`.
3. `ledger.js`: JSON ledger plus MD view; **gate = `/goal` completion fact** (0 gate-blocking active major; author-required is gate-OK and accumulates to the human queue). CLI: init/add/set/count/gate/get/docket/unadjudicated/render.
4. `journal.js`: append-only per-edit revert log (JSONL).
5. `apply-patch.js`: atomic apply plus journal of a drafted patch, and revert (exact-once guard on `before` text).
6. `anchor-diff.js`: locate frozen anchors; flag which `need_audit` when the support region changed.
7. `cross-ref.js`: edit-safety risk pre-filter: does a changed salient token in a patch appear in other passages?
8. `compile-guard.js`: real LaTeX compile (latexmk/pdflatex) or a degraded structural-lint path with `compiled:null` (it reports when it cannot verify).
9. `compliance-check.js`: submission-readiness A: deterministic desk-reject screening.

### Semantic stages (fan-out)

1. `assign-reviewers`: name N subfields, instantiate N domain reviewers from the project gatekeeper core + a generated domain overlay; config-pin / verifier / per-slot degrade headless.
2. `reading-check`: N holistic reviewers each read the WHOLE paper once → weaknesses (significance + kind + verbatim quote; a reviewer that cannot quote the source did not read it) + one overall_confidence + a per-section coverage report; targeted re-invoke mode for anti-skim.
3. `coverage-auditor`: anti-skim L2: flag skimmed (reviewer, section) pairs across the coverage reports.
4. `merge`: semantic dedup across reviewers; the orchestrator derives significance (MAX) / kind (substantive-dominates) / corroboration deterministically after semantic clustering.
5. `trial`: a 5-juror trial tier: whole-paper defense → independent local-context jury (with on-demand context expansion) → a deterministic majority verdict (quorum reached, one side >60%) + a judge that routes a decided-valid charge (valid-fixable vs author-required); escalate to a 12-juror tier only on no clear majority.
6. `polish`: the track that skips the jury: batch copy-edit (mechanical) + batch light-check (minor-substantive); can escalate a misrouted major back to trial.
7. `recall-audit`: Mode A revives wrongly-dropped charges (bias to revive); Mode B spot-checks strong-consensus majors before the edit (guards against the whole panel agreeing on the same mistake).
8. `drafter`: minimal-edit patch for valid-fixable charges.
9. `edit-audit` / `meaning-audit`: the edit-safety semantic half: `edit-audit` checks a risky non-anchor edit (make-sense + cross-section alignment); `meaning-audit` is the four-state frozen-anchor + arc audit.
10. `clerk`: the round boundary: reconcile carried open-questions against this round's edits, dedup re-raises via a deterministic passage_id + similarity merge key, and emit the deterministic convergence counts.

Also present in the Codex contracts: a quick simple 3-lens `review-panel` fast path.

---

## The three primitives: Skill + Fan-out + Memory

1. **Skill (entry point + methodology):** the protocol, the reviewer assignment, the consensus gate, the writing toolkit, the human gates. Detail in `references/review-engine-v3.md`, `references/reviewer-personas.md`, `references/writing-toolkit.md`.

2. **Semantic fan-out engine:** the semantic no-human-in-the-middle steps run through explicitly authorized Codex subagents, using `codex/phase-contracts.md` as the prompt/schema/isolation contract. Simple panel = the `review-panel` fast path; the courtroom engine = `assign-reviewers → reading-check → coverage-auditor → merge → {trial ‖ polish} → recall-audit → drafter → {edit-audit | meaning-audit} → clerk`. The deterministic guards run orchestrator-side via Node: `scripts/` holds `decompose`, `ledger`, `journal`, `apply-patch`, `anchor-diff`, `cross-ref`, `spine`, `compile-guard`, `compliance-check`.

3. **Memory (durable state + learned conventions), two layers:**
   - **Ledger**: `LEDGER.json` resolved at runtime = the machine source of truth, plus a rendered `LEDGER.md` view; managed by `scripts/ledger.js`. The live, mutable issue state across rounds and sessions. Schema plus status state machine: `references/ledger-schema.md`.
   - **Project conventions / host memory**: stable conventions worth recalling next session (this paper's house style, venue, persona tuning).

### Reviewers

The panel is N domain-expert HOLISTIC reviewers (default 3, range 2-4), assigned at runtime to the paper's subfields, all sharing a senior-reviewer gatekeeper core (harsh, precise, constructive; separate fatal flaws from fixable nits; reason across sections). When a reviewer slot cannot be confirmed (headless, unverifiable), that slot degrades to a generic gatekeeper (one bad slot never degrades the whole panel); the generic fallback lenses are:

- **Theory / Foundations**: definitions, proof gaps, notation, invariance/optimality/generality claims.
- **Empirical / Benchmark**: baseline fairness/vintage, metric correctness, dataset splits, variance, ablation coverage, cherry-picking.
- **Applied / Systems**: practicality, efficiency/latency/memory claims, reproducibility, deployment realism, scaling.

(These are an unordered tendency, not fixed slots; reviewer IDs `R1..RN` are positional, assigned by subfield order.)

The writing toolkit names (prompt bodies not shown here): `translate-to-english`, `polish-english`, `de-ai`, `compress`, `expand`, `caption`, `experiment-analysis`, `logic-check`.

---

## The six hard rules

1. **Never edit the manuscript without explicit author sign-off.** Auto-mode carve-out: the rule HOLDS; auto satisfies it via UP-FRONT sign-off (the `spine` + reviewer-assignment confirmation plus the pre-authorized bounded-aggressive policy) plus the return queue, not per-edit sign-off.
2. **Reviewers / jurors are isolated.** Fresh eyes per round: no cross-talk, no prior-round leakage, no sight of the `ledger`. Enforced by (a) what goes into each agent's prompt AND (b) an explicit ISOLATION instruction in every reviewer-type prompt.
3. **Every valid-fixable issue carries a `close_criterion`** (one concrete sentence describing what an edit must satisfy), set by the judge.
4. **No leakage into the reviewed text.** Revision logs, back-translations, and self-check verdicts are author-side aids; they never enter the manuscript or any frozen snapshot.
5. **Disagreement resolves through discussion, then override (logged), never a silent dismissal.**
6. **No hardcoded paths or project files in the skill.** Resolve at runtime.

---

## Architecture notes

- Codex does not execute Claude Workflow files. Semantic phases are defined in `codex/phase-contracts.md`; deterministic guards run orchestrator-side via Node.
- `compile-guard.js` is explicit about what it cannot verify: when it cannot truly compile, it degrades to structural lint and reports `compiled:null`.
- Submission-readiness is cross-mode, two parts: **A** = `compliance-check.js` plus a semantic agent; **B** = a compile-driven layout loop reusing `compile-guard.js` plus Read-on-PDF.

Your project files, ledger, journal, and patches stay inside your local paper project. PaperJury has no backend or server of its own, so nothing is sent to a PaperJury server. The review runs through your own Codex session, which means the model itself may run in the cloud: how your content is handled there follows the terms and settings of that host environment, not anything PaperJury adds on top.

---

## Project structure

| Path | Purpose |
|---|---|
| `.codex-plugin/` | Codex plugin marketplace metadata. |
| `skills/` | The `paperjury` skill exposed to Codex. |
| `codex/` | Runtime mapping, phase contracts, and agent driving guide for Codex. |
| `agents/` | Semantic agent definitions used by the Codex port. |
| `scripts/` | Deterministic guards: ledger, journal, apply-patch, anchor-diff, cross-ref, compile-guard, doctor, and related checks. |
| `references/` | Engine protocol, ledger schema, reviewer personas, writing toolkit, and methodology notes. |
| `docs/` | Interactive overview, site assets, and design-entry docs. |
| [dogfood sample](https://github.com/u7079256/paperjury-codex/tree/main/samples/dogfood) | Repo-level before/after PDFs and a human-verified run report. |

---

</details>

## Roadmap

Where this is going (planned, not yet shipped):

- **Reviewer personas tuned to each venue community's taste.** CVPR, ACL, and NeurIPS reviewers do not critique the same way; the goal is a reviewer that carries each community's expectations, beyond the current three-family style context.
- **Vision-based layout verification**: compile, render, and check the visual layout (column overflow, figure placement), not just the compile log.
- **Automatic venue detection** from your `.cls` / template.
- **Validation of the engine on real papers at scale.**

---

## File and path reference

- Engine protocol (every orchestrator seam): `references/review-engine-v3.md`
- Codex runtime mapping: `codex/runtime.md`
- Codex semantic phase contracts: `codex/phase-contracts.md`
- Auto protocol: `references/auto-mode.md`
- Personas / writing toolkit: `references/reviewer-personas.md`, `references/writing-toolkit.md`
- Ledger schema + status machine: `references/ledger-schema.md`
- Submission compliance: `references/submission-compliance.md`
- Codex runtime guide: `codex/AGENT-GUIDE.md`
- Codex agent guide: `codex/AGENT-GUIDE.md`
- Scripts dir: `scripts/` (decompose, ledger, journal, apply-patch, anchor-diff, cross-ref, spine, compile-guard, compliance-check)
- Codex runtime package: `codex/`

---

## Credits

The spine and anti-drift design (the anchor logic-transfer audit, the claim register, and the minimal-edit, intent-preserving revision policy) is inspired by [PaperSpine](https://github.com/WUBING2023/PaperSpine), a motivation-driven paper drafting and rewriting skill. PaperSpine is a forward generate/rewrite tool with no adversarial loop; PaperJury borrows its anchoring idea and its "deterministic scripts for checkable steps, model agents for judgment" mechanism, then adds the adversarial courtroom review engine on top.
