# `/craft`

A structured seven-phase development pipeline — Explore, Clarify, Architect, Implement, Review,
Refine, Push — that takes an issue or task description and produces a draft pull request.

## Design Principles

**Gates at decisions, not at checkpoints.** Human gates appear after Clarify (before any design
work), after Architect (before any code), in Refine (for triage decisions), and in Push (before the
PR opens). These are the points where human judgment changes the outcome. Phases that produce no
human-visible decision — Explore, Implement, Review — run without a gate.

**Review drives quality.** Focused reviewers work in parallel across distinct domains,
cross-corroborate findings, independently verify uncertain results, and feed structured refinement.
Review drives quality; everything before it produces something worth reviewing.

**Polyglot by design.** The pipeline is opinionated about process, not about stack. It carries no
language-specific rules or framework assumptions.

## Install

```
/plugin marketplace add sargunv/claude-plugins
/plugin install craft@sargunv-plugins
```

## Usage

[`/craft <task>`](skills/craft/SKILL.md) runs the full pipeline from Explore through Push. Each
phase can also be invoked individually:

1. [`/craft-explore <task>`](skills/craft-explore/SKILL.md): Deep codebase exploration that spawns
   parallel sub-agents across architecture, change surface, data, dependencies, and PR history.
2. [`/craft-clarify <task>`](skills/craft-clarify/SKILL.md): Turns a vague task into numbered
   requirements (R1..Rn) by asking targeted questions with concrete recommendations.
3. [`/craft-architect <workpad>`](skills/craft-architect/SKILL.md): Designs an implementation plan,
   stress-tests it with an adversarial sub-agent, then presents a single recommendation.
4. [`/craft-implement <workpad>`](skills/craft-implement/SKILL.md): Executes an approved
   architecture plan, formalizes acceptance criteria, implements changes, and runs linters/tests.
5. [`/craft-review [branch]`](skills/craft-review/SKILL.md): Multi-reviewer code review that selects
   reviewers from an inline topic map, spawns them in parallel, corroborates findings, verifies
   acceptance criteria, and reports.
6. [`/craft-refine [branch]`](skills/craft-refine/SKILL.md): Turns review findings into clean code —
   applies auto-fixes, batches remaining findings for human triage, re-runs narrow review on fix
   deltas.
7. [`/craft-push [workpad]`](skills/craft-push/SKILL.md): Pushes the branch and opens a draft PR/MR
   using whatever forge tooling is available (CLI or MCP), then handles wrap-up (filing deferred
   findings in your issue tracker, workpad cleanup).

### Standalone skills

[`/prose [file|glob|description]`](skills/prose/SKILL.md) — Write or edit prose for human readers
(docs, READMEs, changelogs, error messages, UI copy) using Strunk's _Elements of Style_. Not part of
the craft pipeline; invoke directly when prose work is the task.

### The workpad

The `workpad.md` records the task description, exploration report, requirements, architectural
decision, acceptance criteria, and phase log. Pass it to any phase skill to resume or re-run from
that point:

```
/craft-architect workpad.md
/craft-implement workpad.md
```

### Review

A broad set of focused reviewers — covering correctness, security, performance, concurrency,
accessibility, memory safety, and more — are selected by the inline topic map in
[craft-review](skills/craft-review/SKILL.md) based on what the diff touches. That file is also the
place to look when adding a new reviewer: register the agent in `agents/` and add a bullet to the
topic map.

### Rework

If the human requests a structural change or adds a requirement after review, the rework protocol
creates a new branch (`<branch-name>-v2`), renames the old workpad to `workpad-v1.md`, creates a
fresh workpad with the old requirements plus any new ones, and returns to `/craft-architect`. The
old workpad is preserved for reference.

### Abandoning a branch

Close any open PR/MR, delete the branch, and discard or archive the workpad. File deferred items
before discarding.

## Acknowledgments

Built on the foundation of Anthropic's
[feature-dev](https://github.com/anthropics/claude-code/tree/main/plugins/feature-dev) plugin, with
ideas drawn from [compound-engineering](https://github.com/EveryInc/compound-engineering-plugin),
[superpowers](https://github.com/obra/superpowers), [spec-kit](https://github.com/github/spec-kit),
[symphony](https://github.com/openai/symphony), and
[the-elements-of-style](https://github.com/obra/the-elements-of-style).
