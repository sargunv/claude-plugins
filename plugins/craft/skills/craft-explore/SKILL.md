---
name: craft-explore
description: Use at the start of the craft pipeline, or standalone when exploring an unfamiliar codebase area before making design decisions. Spawns parallel sub-agents across architecture, change surface, data, dependencies, and PR history to build a dense factual picture of the codebase.
argument-hint: "[issue URL, ID, or task description]"
allowed-tools: Agent Bash Read Glob Grep WebFetch WebSearch
---

# /craft-explore — Codebase Exploration

Arguments: $ARGUMENTS

Git context: !`git log --oneline -5 2>/dev/null || echo "(not a git repo)"`

---

## Role

Orchestrate exploration to build a dense, accurate picture of the codebase from multiple independent
angles simultaneously. Dense signal here prevents wasted cycles in downstream work (clarification,
architecture, or direct implementation).

## Explorer Agents

Spawn explorer sub-agents **in parallel** using the Agent tool. Each receives a distinct,
non-overlapping focus.

**Always spawn:**

**Agent A — Architecture & Entry Points.** Focus: module/package organization, main entry points
(routes, CLI commands, event handlers, main functions), core abstractions, dependency
injection/service wiring, layering. Does NOT cover: specific data flows, test patterns, external
deps.

**Agent B — Change Surface.** Focus: files and interfaces directly touched by this task, their
callers, their tests, types/interfaces involved, recent commits to affected files. Does NOT cover:
overall architecture, external deps.

**Agent C — Commit / PR History.** Focus: prior commits that touched the change surface, explicit
references to past PRs in the task, reverts and follow-ups, reasons an adjacent design was rejected
previously. Uses version-control history (`git log`, host-specific PR tooling when available).

**Spawn when warranted by the task:**

**Agent D — Data & State.** Focus: data models, database schema, state management, persistence
layer, migrations. Spawn when the task involves database changes, new models, or state management.

**Agent E — Dependencies & Integration.** Focus: external libraries used, API contracts, environment
variables, config files, third-party service integrations. Spawn when the task involves new
dependencies or external service changes.

## Invocation Prompt for Each Agent

Pass each sub-agent:

- The task description (`$ARGUMENTS`) and any available issue body.
- Its assigned focus area (verbatim from above).
- An instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/craft-explore/explorer-contract.md` for the
  required output format and rules.

## Orchestrator Instructions

1. Launch all selected agents **in parallel** using the Agent tool.
2. Wait for all agents to return.
3. Read the key files each agent surfaced **before synthesizing** — do not summarize agent
   summaries; read the actual files.
4. Produce the **Exploration Report** below. This is YOUR synthesis across all agents' outputs —
   distill the most signal-dense facts and cross-reference where agents surfaced the same files.

## Exploration Report Format

```markdown
## Exploration Report

### Key Facts (≤20 bullets — most actionable first)

- [file:line] [specific, dense fact — no vague generalizations]
- ...

### Code Snippets (exact code, not pseudocode)

**[Purpose label]** — `path/to/file.ts:42`

\`\`\` [exact code snippet] \`\`\`

### Key Files

- `path/to/file.ts` — [purpose, why it matters for this task]
- ...
```

The full agent outputs are available in the live conversation context. If key signal is missing from
the synthesis, trace back to the specific agent's output.

## Rules

- Every Key Facts bullet must be specific with a file reference.
- Code Snippets must be actual code — if not found, say so explicitly.
- Key Files list drives downstream work — be precise.
- Do not include implementation recommendations — that is the architect's job.

## Workpad Update

After producing the Exploration Report, write it to `workpad.md` under `## Exploration Report`. If
no workpad exists, skip this step (workpad is created by `/craft`; standalone explore invocations
may not have one).
