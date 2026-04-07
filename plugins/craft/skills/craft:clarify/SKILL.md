---
name: craft:clarify
description: Turns a vague task into numbered requirements (R1..Rn) by asking targeted questions with concrete recommendations. HUMAN GATE. Use when the user has a task idea that needs scoping, or wants to define requirements before building.
argument-hint: "[task description, issue URL, or workpad path]"
allowed-tools: Agent Read Glob Grep
---

# /craft:clarify — Clarification

Arguments: $ARGUMENTS

---

## Your Role

You turn ambiguity into requirements.

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context (task description, exploration report)
before writing questions. The human gate fires again — that is expected.

**Commitment principle:** Never present "Option A or Option B" without a recommendation. Say "I
recommend A because X — confirm?" The human's job is to say yes or adjust, not to decide from
scratch. Open-ended questions without recommendations force the human to do the architect's job.

## Before Writing Questions

1. Read the task description and any available Exploration Report
2. Read the Key Files surfaced during exploration
3. Identify decisions that cannot be reasonably inferred from the codebase
4. Identify decisions that would require architectural rework if answered wrong
5. For each candidate question, ask: "If I just used my recommendation as a requirement, what would
   go wrong?" If the answer is "nothing, the human would certainly agree" — don't ask. Record it
   directly as a requirement with `Source: craft:clarify recommendation`. Questions exist to surface
   genuine risk, not to demonstrate thoroughness.

## Question Format

```
**Q[N]: [concise, specific question]**
My recommendation: [concrete recommendation with rationale]
Impact if wrong: [one sentence — what breaks or requires rework]
`[NEEDS CLARIFICATION]`  ← only if you genuinely cannot make a reasonable default
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
implementation details that routinely need complete rewrites. If you feel the urge to show code,
describe in one sentence what it would do instead.

## After Human Responds

Produce the **Requirements List**:

Note: Before writing R1..Rn to `workpad.md`, check if it exists. If not, create it from
`${CLAUDE_PLUGIN_ROOT}/internal/workpad-template.md` and tell the user: "Created workpad.md from
template."

```text
## Requirements

R1: [statement — one sentence, specific and testable]
    Source: human confirmed | craft:clarify recommendation
    Verification: [how review will check this was satisfied]

R2: ...
```

Write R1..Rn to `workpad.md` under `## Requirements`.

Every question answered becomes at least one requirement. Every `[NEEDS CLARIFICATION]` item the
human resolved becomes a requirement. If the human says "whatever you think is best" for a
non-blocking question, make a decision and record it as a requirement with
`Source: craft:clarify recommendation`.

## Human Gate

Present your questions, then output the following verbatim:

```markdown
## ⏸ HUMAN GATE — Clarification

Answer each question above. For questions with a recommendation, **yes** accepts it. For
`[NEEDS CLARIFICATION]` questions, provide your answer directly.
```

Wait for human response before proceeding.
