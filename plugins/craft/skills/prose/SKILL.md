---
name: prose
description: Use when writing or editing prose for human readers — docs, READMEs, changelogs, migration guides, error messages, UI copy. Applies Strunk's Elements of Style. Not part of the craft pipeline; standalone only.
argument-hint: "[file path, glob pattern, or writing task description]"
allowed-tools: Agent Read Glob Grep
---

# /prose — Write and Edit Prose

Arguments: $ARGUMENTS

---

## Role

Write or edit prose for human readers. This skill dispatches the work to the **prose** sub-agent so
the ~12,000-token style guide stays out of the orchestrator's context.

## Dispatch

For each file (or each logical prose deliverable), spawn a prose sub-agent using the Agent tool with
`subagent_type: "craft:prose"`. Pass it:

- The task — a file path, glob pattern, or description of prose to write.
- Any relevant context from the caller (audience, purpose, voice to preserve).

For edits across multiple files, spawn one prose agent per file **in parallel**, unless the required
edits are small _and_ closely coupled. The prose agent reads the style guide, writes or edits the
prose, and returns.
