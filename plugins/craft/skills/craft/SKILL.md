---
name: craft
description: End-to-end development pipeline (explore → clarify → architect → implement → review → refine) that takes a task from idea to PR. Use when the user asks to build a feature, fix a bug, implement an issue, or work on a task that needs planning, coding, and review. Accepts issue URLs, issue IDs, or plain descriptions.
argument-hint: "[issue URL, issue ID, or task description]"
allowed-tools: Agent Bash Read Write Edit Glob Grep
---

# /craft — Full Development Pipeline

Arguments: $ARGUMENTS

---

## Setup

Create `workpad.md` in the project root from the template at
`${CLAUDE_PLUGIN_ROOT}/internal/workpad-template.md`. If a workpad already exists, read it and
resume from the current phase in the Phase Log.

If `$ARGUMENTS` is an issue URL or ID, fetch the full issue using the appropriate tool (e.g.,
`gh issue view`, Linear MCP, Jira MCP, or whatever the project uses) and write it as the task
description in the workpad header.

## Progress Tracking

Before starting the pipeline, create a task for each phase with TaskCreate:

- "Explore codebase" (activeForm: "Exploring codebase")
- "Clarify requirements" (activeForm: "Clarifying requirements")
- "Design architecture" (activeForm: "Designing architecture")
- "Implement changes" (activeForm: "Implementing changes")
- "Polish prose" (activeForm: "Polishing prose") — only if prose step runs
- "Review implementation" (activeForm: "Reviewing implementation")
- "Refine and open PR" (activeForm: "Refining and opening PR")

Mark each task `in_progress` when a phase starts and `completed` when it finishes. The top-level
orchestrator owns task state; sub-skills do not manage tasks.

## Pipeline

Execute each step in order. Wait for an explicit human response at every HUMAN GATE. If a phase
errors, report it and ask whether to retry, skip, or abort.

1. `/craft:explore $ARGUMENTS`

   Writes the Exploration Report (key facts, code snippets, key files) to the workpad. Update Phase
   Log: explore → done. No human gate.

2. `/craft:clarify workpad.md`

   Reads the exploration report, asks targeted questions, and produces requirements R1..Rn. HUMAN
   GATE. After response, record answers and requirements in the workpad. Update Phase Log: clarify →
   done.

3. `/craft:architect workpad.md`

   Reads requirements and exploration report, runs parallel architecture agents, and synthesizes a
   recommended approach. HUMAN GATE. After approval, record the chosen approach in the workpad.
   Update Phase Log: architect → done.

4. `/craft:implement workpad.md`

   Creates a branch, formalizes acceptance criteria, implements the approved plan, and runs
   linters/tests. Update Phase Log: implement → done. No human gate.

4b. _(Optional)_ `/craft:prose workpad.md`

If the requirements or plan include prose deliverables (documentation, READMEs, changelogs,
migration guides, error messages, UI copy), run this phase. Implement handles code; prose handles
human-facing text. Skip if the task has no prose deliverables. Update Phase Log: prose → done if
run, prose → skipped if not.

5. `/craft:review`

   Topic-tags the diff, spawns targeted reviewers, verifies acceptance criteria, and produces a
   scored findings report. Update Phase Log: review → done. No human gate.

6. `/craft:refine`

   Applies auto-fixes, batches remaining findings for human triage, and opens the PR. HUMAN GATE
   before opening the PR.
