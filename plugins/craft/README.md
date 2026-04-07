# `/craft`

A structured six-phase development pipeline — Explore, Clarify, Architect, Implement, Review, Refine
— that takes an issue or task description and produces a reviewed, mergeable pull request.

## Design Principles

**Gates at decisions, not at checkpoints.** Human gates appear after Clarify (before any design
work), after Architect (before any code), and in Refine (before the PR opens). These are the three
points where human judgment changes the outcome. Phases that produce no human-visible decision —
Explore, Implement, Review: run without a gate.

**Review drives quality.** Focused reviewers work in parallel across non-overlapping domains,
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

[`/craft <task>`](skills/craft/SKILL.md) runs the full pipeline from Explore through Refine. Each
phase can also be invoked individually:

1. [`/craft:explore <task>`](skills/craft:explore/SKILL.md): Deep codebase exploration that spawns
   parallel sub-agents across architecture, change surface, data, and dependencies.
2. [`/craft:clarify <task>`](skills/craft:clarify/SKILL.md): Turns a vague task into numbered
   requirements (R1..Rn) by asking targeted questions with concrete recommendations.
3. [`/craft:architect <workpad>`](skills/craft:architect/SKILL.md): Designs an implementation plan
   by running parallel architecture agents, then stress-tests the synthesis adversarially.
4. [`/craft:implement <workpad>`](skills/craft:implement/SKILL.md): Executes an approved
   architecture plan, creates a branch, formalizes acceptance criteria, implements changes, and runs
   linters/tests.
   - [`/craft:prose [file|workpad]`](skills/craft:prose/SKILL.md): Writes or edits prose for
     clarity, concision, and correctness using Strunk's _Elements of Style_.
5. [`/craft:review [branch]`](skills/craft:review/SKILL.md): Multi-reviewer code review that
   topic-tags the diff, spawns targeted reviewers in parallel, verifies acceptance criteria, and
   scores findings.
6. [`/craft:refine [branch]`](skills/craft:refine/SKILL.md): Turns review findings into a mergeable
   PR, applies auto-fixes, batches remaining findings for human triage, then opens the PR.

### The workpad

The `workpad.md` records the task description, exploration report, requirements, architectural
decision, acceptance criteria, and phase log. Pass it to any phase skill to resume or re-run from
that point:

```
/craft:architect workpad.md
/craft:implement workpad.md
```

### Review

A broad set of focused reviewers: covering correctness, security, performance, concurrency,
accessibility, and more: are activated by topic tags on each diff. See the
[topic → reviewer map](skills/craft:review/topic-reviewer-map.md) for the full list, their domains,
and activation rules. To add a reviewer, see [Adding a Reviewer](docs/adding-a-reviewer.md).

### Rework

If the human requests a structural change or adds a requirement after review, the rework protocol
creates a new branch (`<branch-name>-v2`), renames the old workpad to `workpad-v1.md`, creates a
fresh workpad with the old requirements plus any new ones, and returns to `/craft:architect`. The
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
