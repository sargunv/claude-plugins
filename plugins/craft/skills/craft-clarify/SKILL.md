---
name: craft-clarify
description: This skill should be used when the user asks to "clarify requirements", "scope a task", "define requirements", "what should we build", or has a vague task idea that needs scoping before implementation. Turns a vague task into numbered requirements (R1..Rn) by asking targeted questions with concrete recommendations. HUMAN GATE.
argument-hint: "[task description, issue URL, or workpad path]"
---

# /craft-clarify — Clarification

Arguments: $ARGUMENTS

---

## Role

Turn ambiguity into requirements.

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context (task description, exploration report)
before writing questions. The human gate fires again — that is expected.

**Commitment principle:** Never present "Option A or Option B" without a recommendation. Say "I
recommend A because X — approve?" The human's job is to approve or adjust, not to decide from
scratch. Open-ended questions without recommendations force the human to do the architect's job.

## Before Writing Questions

1. Read the task description and any available Exploration Report. If no exploration report exists
   (standalone invocation), explore the relevant codebase area directly — read entry points, key
   modules, and related files to build enough context for informed questions.
2. Read the Key Files surfaced during exploration (or discovered above)
3. Identify decisions that cannot be reasonably inferred from the codebase
4. Identify decisions that would require architectural rework if answered wrong
5. For each candidate question, ask: "If I just used my recommendation as a requirement, what would
   go wrong?" If the answer is "nothing, the human would certainly agree" — don't ask. Record it
   directly as a requirement with `Source: craft-clarify recommendation`. Questions exist to surface
   genuine risk, not to demonstrate thoroughness.

## Question Format

```
**Q[N]: [concise, specific question]**
My recommendation: [concrete recommendation with rationale]
Impact if wrong: [one sentence — what breaks or requires rework]
`[NEEDS CLARIFICATION]`  ← only when no reasonable default can be determined
```

Rules:

- **Maximum 7 questions**. More than 7 means you're over-clarifying. Prune ruthlessly.
- **Maximum 3 `[NEEDS CLARIFICATION]` markers** total across all questions. Everything else must
  have a recommendation.
- Order: blockers first, important decisions second, advisory last.
- Never ask about things answerable from the codebase.
- Never ask about preferences without offering a concrete default.

**ABSOLUTE CONSTRAINT: Do not write any code during this phase.** Not as examples, rough sketches,
or pseudocode. Write English descriptions only. Code written during Clarify anchors prematurely on
implementation details that routinely need complete rewrites. Instead of showing code, describe in
one sentence what it would do.

## After Human Responds

Produce the **Requirements List**:

Note: Before writing R1..Rn to `workpad.md`, check if it exists. If not, create it from
`../../references/workpad-template.md` relative to this skill file and tell the user: "Created
workpad.md from template."

```text
## Requirements

R1: [statement — one sentence, specific and testable]
    Source: human confirmed | craft-clarify recommendation
    Verification: [how review will check this was satisfied]

R2: ...
```

Write R1..Rn to `workpad.md` under `## Requirements`.

Every question answered becomes at least one requirement. Every `[NEEDS CLARIFICATION]` item the
human resolved becomes a requirement. If the human says "whatever you think is best" for a
non-blocking question, decide and record it as a requirement with
`Source: craft-clarify recommendation`.

## Human Gate

Present the questions, then output the following verbatim:

```markdown
## ⏸ HUMAN GATE — Clarification

Answer each question above. For questions with a recommendation, **approve** accepts it. For
`[NEEDS CLARIFICATION]` questions, provide an answer directly.
```

Wait for human response before proceeding.
