---
name: craft-explore
description: This skill should be used when the user asks to "explore the codebase", "how does this work", "what would this change touch", "map out the code", or wants to understand a codebase area before making decisions. Deep codebase exploration that spawns parallel sub-agents across architecture, change surface, data, and dependencies.
argument-hint: "[issue URL, ID, or task description]"
---

# /craft-explore — Codebase Exploration

Arguments: $ARGUMENTS

---

## Role

Orchestrate exploration to build a dense, accurate picture of the codebase from multiple independent
angles simultaneously. Dense signal here prevents wasted cycles in downstream work (clarification,
architecture, or direct implementation).

## Supporting files

- [exploration-pass-contract.md](exploration-pass-contract.md) — shared output and behavior contract
  for exploration sub-agents

## Explorer Passes

Spawn the following exploration sub-agents **in parallel**. Each has a distinct, non-overlapping
focus. Tell each sub-agent what the others are NOT covering.

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

1. Determine which sub-agents to spawn (always A+B, add C/D when warranted by the task).
2. Launch all selected sub-agents **in parallel**. For each one, pass:
   - The task description ($ARGUMENTS)
   - Their specific focus area
   - Explicit statement of what the OTHER sub-agents are covering (to prevent overlap)
   - Read and follow `exploration-pass-contract.md` next to this skill file
3. Wait for all sub-agents to return.
4. Read the key files each sub-agent surfaced **before synthesizing** — do not summarize agent
   summaries; read the actual files.
5. Produce the **Exploration Report** below. This is YOUR synthesis across all sub-agents' outputs —
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
- Code Snippets must be actual code — if not found, say so explicitly
- Key Files list drives downstream work — be precise
- Do not include implementation recommendations — that is the architect's job
- No human gate at the end of this phase

## Workpad Update

After producing the Exploration Report, write it to `workpad.md` under `## Exploration Report`. If
no workpad exists, skip this step (workpad is created in Phase 0; standalone explore invocations may
not have one).
