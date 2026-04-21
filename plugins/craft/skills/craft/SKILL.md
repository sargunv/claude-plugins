---
name: craft
description: This skill should be used when the user asks to "build a feature", "fix a bug", "implement an issue", "work on a task", or provides an issue URL/ID with intent to plan, code, and review. End-to-end development pipeline (explore → clarify → architect → implement → review → refine → push) that takes a task from idea to draft PR.
argument-hint: "[issue URL, issue ID, or task description]"
allowed-tools: Agent Bash Read Write Edit Glob Grep
---

# /craft — Full Development Pipeline

Arguments: $ARGUMENTS

---

## Setup

Create `workpad.md` in the project root from the template at
`${CLAUDE_PLUGIN_ROOT}/skills/craft/workpad-template.md`. If a workpad already exists, read it and
resume from the current phase in the Phase Log.

If `$ARGUMENTS` is an issue URL or ID, fetch the full issue using the appropriate tool (e.g.,
`gh issue view`, Linear MCP, Jira MCP, or whatever the project uses) and write it as the task
description in the workpad header.

## Progress Tracking

Before starting the pipeline, create a todo item for each phase. Use the literal skill names
verbatim, to remind you to use the proper skill for each step:

- "/craft-explore"
- "/craft-clarify"
- "/craft-architect"
- "/craft-implement"
- "/craft-review"
- "/craft-refine"
- "/craft-push"

Mark each task `in_progress` when a phase starts and `completed` when it finishes. You own task
state; sub-skills do not manage tasks.

## Pipeline

Execute each step in order. Wait for an explicit human response at every HUMAN GATE. If a phase
errors, report it and ask whether to retry, skip, or abort.

1. `/craft-explore $ARGUMENTS`, then update Phase Log: explore → done. No human gate.

2. `/craft-clarify workpad.md` includes a HUMAN GATE. After response, record answers and
   requirements in the workpad. Update Phase Log: clarify → done.

3. `/craft-architect workpad.md` includes a HUMAN GATE. After approval, record the chosen approach
   in the workpad. Update Phase Log: architect → done.

4. `/craft-implement workpad.md`, then update Phase Log: implement → done. No human gate.

5. `/craft-review workpad.md`, then update Phase Log: review → done. No human gate.

6. `/craft-refine workpad.md` includes a HUMAN GATE for triage. After it returns, the branch has
   auto-fixes and triaged fixes applied, and deferred items are recorded in the workpad. Update
   Phase Log: refine → done.

7. `/craft-push workpad.md` includes a HUMAN GATE for the PR body. Opens the draft PR and handles
   wrap-up (deferred findings, workpad cleanup). Update Phase Log: pr → done.
