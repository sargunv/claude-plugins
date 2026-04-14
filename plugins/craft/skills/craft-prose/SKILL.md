---
name: craft-prose
description: This skill should be used when the user asks to "write documentation", "edit the README", "polish this text", "write a migration guide", "improve error messages", or needs to write, rewrite, or polish any prose meant for human readers (docs, changelogs, UI copy, commit messages). Also activates as an optional pipeline step when a task involves prose alongside code. Uses Strunk's Elements of Style.
argument-hint: "[file path, glob pattern, or writing task description]"
---

# /craft-prose — Write and Edit Prose

Arguments: $ARGUMENTS

---

## Style Reference

`elements-of-style.md` next to this skill file contains Strunk's _The Elements of Style_ (1918): 18
rules of usage and composition, plus a reference of commonly misused words.

## Determining the Task

If `$ARGUMENTS` is a file path or glob pattern, edit existing prose in those files. If it is a
description (e.g., "write a migration guide for v3"), write new prose. If running as a pipeline
step, the workpad identifies which deliverables need prose.

## Workflow

1. Read the target file(s) or understand the audience and purpose from context.
2. Write or edit the prose. Preserve the author's voice and intent — the goal is clarity, not
   uniformity. Do not rewrite prose that is already clear.
3. After writing, re-read the full file to check flow and consistency.

## Pipeline Integration

When invoked from `/craft` alongside implementation, prose is the authoring step — it writes the
human-facing text that the task requires. Implement handles code; prose handles docs, READMEs,
changelogs, migration guides, error messages, and any other prose deliverable.

1. Read the workpad. Identify which requirements or plan items need prose deliverables.
2. For each deliverable, write or edit following the workflow above.
3. Tighten any prose the implement phase wrote incidentally (doc comments, error strings).
4. Commit prose work as a separate commit.

## Sub-agents

Always dispatch sub-agents for prose work, even for a single file. The style guide is ~12,000 tokens
— keeping that out of the orchestrator's context matters. Dispatch one sub-agent per file. Each
sub-agent reads `elements-of-style.md` next to this skill file and one file, writes or edits the
prose, and returns.
