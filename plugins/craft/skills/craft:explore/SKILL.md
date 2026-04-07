---
name: craft:explore
description: Deep codebase exploration that spawns parallel sub-agents across architecture, change surface, data, and dependencies. Use when the user wants to understand how something works, map out what a change would touch, or research a codebase area before making decisions.
argument-hint: "[issue URL, ID, or task description]"
allowed-tools: Agent Read Glob Grep WebFetch WebSearch
---

# /craft:explore — Codebase Exploration

Arguments: $ARGUMENTS

Git context: !`git log --oneline -5 2>/dev/null || echo "(not a git repo)"`

---

## Your Role

You are the exploration orchestrator. Build a dense, accurate picture of the codebase from multiple
independent angles simultaneously. Output feeds the Clarify phase — dense signal here prevents
wasted architecture cycles later.

## Explorer Agents

Spawn the following sub-agents **in parallel** using the Agent tool. Each has a distinct,
non-overlapping focus. Tell each agent what the others are NOT covering.

**Always spawn (required):**

**Agent A — Architecture & Entry Points** Focus: module/package organization, main entry points
(routes, CLI commands, event handlers, main functions), core abstractions, dependency
injection/service wiring, layering. Does NOT cover: specific data flows, test patterns, external
deps.

**Agent B — Change Surface** Focus: files and interfaces directly touched by this task, their
callers, their tests, types/interfaces involved, recent commits to affected files. Does NOT cover:
overall architecture, external deps.

**Additional agents (spawn when warranted by the task):**

**Agent C — Data & State** Focus: data models, database schema, state management, persistence layer,
migrations. Does NOT cover: entry points, test patterns. Spawn when the task involves database
changes, new models, or state management.

**Agent D — Dependencies & Integration** Focus: external libraries used, API contracts, environment
variables, config files, third-party service integrations. Does NOT cover: internal code structure.
Spawn when the task involves new dependencies or external service changes.

## Instructions

1. Determine which agents to spawn (always A+B, add C/D when warranted by the task).
2. Launch all selected agents **in parallel** using the Agent tool with
   `subagent_type: "craft:explorer"`. Pass to each agent's prompt:
   - The task description ($ARGUMENTS)
   - Their specific focus area
   - Explicit statement of what the OTHER agents are covering (to prevent overlap)
3. Wait for all agents to return. Each agent produces Key Facts, Code Snippets, Key Files, and
   Appendix.
4. Read the key files each agent surfaced **before synthesizing** — do not summarize agent
   summaries; read the actual files.
5. Produce the **Exploration Report** below. This is YOUR synthesis across all agents' outputs —
   distill the most signal-dense facts, cross-referencing where agents surfaced the same files.
   - The `Appendix: Full Agent Outputs` section contains each agent's raw output for traceability.

## Exploration Report Format

```markdown
## Exploration Report

### Key Facts (≤20 bullets — most actionable first)

- [file:line] [specific, dense fact — no vague generalizations]
- ...

### Code Snippets (exact code, not pseudocode)

**[Purpose label]** — `path/to/file.ts:42`
```

[exact code snippet]

```
### Key Files
- `path/to/file.ts` — [purpose, why it matters for this task]
- ...
```

The full agent outputs are available in the live conversation context. If key signal is missing from
the synthesis, trace back to the specific agent's output.

## Rules

- Every Key Facts bullet must be specific with a file reference
- Code Snippets must be actual code — if you cannot find it, say so explicitly
- Key Files list is what the Clarify/Architect phases will read — be precise
- Do not include implementation recommendations — that is the architect's job
- No human gate at the end of this phase

## Workpad Update

After producing the Exploration Report, write it to `workpad.md` under `## Exploration Report`. If
no workpad exists, skip this step (workpad is created in Phase 0; standalone explore invocations may
not have one).
