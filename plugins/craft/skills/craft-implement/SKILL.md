---
name: craft-implement
description: Executes an approved architecture plan — creates a branch, formalizes acceptance criteria, implements changes, and runs linters/tests. Requires requirements (R1..Rn) and an approved architectural approach from a workpad or inline context. Defers out-of-scope discoveries. Typically invoked by the /craft pipeline after architecture approval.
argument-hint: "[workpad path or inline approved approach]"
allowed-tools: Agent Bash Read Write Edit Glob Grep WebFetch WebSearch
---

# /craft-implement — Implementation

Arguments: $ARGUMENTS

---

## Role

Execute the approved plan precisely. Do not improve beyond scope. Do not refactor things noticed in
passing. Do not add features that were not approved.

**Out-of-scope discipline — this is non-negotiable:** Every improvement noticed outside the approved
plan goes into a `[DEFERRED]` block in the workpad. Never implement it. Scope expansion during
implementation is the top cause of review findings, test failures, and extended review cycles. If a
pre-existing bug unrelated to this task is discovered, log it as `[PRE-EXISTING]` and continue. Do
not fix it.

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context. If the branch already exists and is
checked out, skip branch creation and continue from where implementation stopped.

## Setup

1. Read `workpad.md` (or `$ARGUMENTS`). If the workpad is missing or incomplete, stop and ask for
   it. Verify it contains:
   - Requirements R1..Rn
   - Approved architectural approach
   - Files to create/modify
   - Test strategy

2. Fill in workpad `## Implementation Plan` header fields: branch name, started timestamp, and base
   commit (`git rev-parse HEAD`).

## Acceptance Criteria (Before Writing Any Code)

Before writing a single line of implementation, formalize the acceptance criteria. Derive them from
R1..Rn and the approved architecture.

Write to workpad under `## Acceptance Criteria`:

```
AC1: [from R1] [specific, testable condition]
     Auto-verify: [test <name> | grep <pattern> in <file> | lint rule <rule> | [MANUAL-VERIFY]]

AC2: [from R2] [...]
```

Every requirement Rn must map to at least one AC. If no auto-verify method can be written, mark it
`[MANUAL-VERIFY]` — it will be flagged for human verification in review.

After implementing each AC, add a completion note inline:

```
AC1: [criterion]  ✓ DONE — [one sentence: how verified, e.g. "test passes", "grep confirms", "manually checked"]
```

This is what completion bar item 4 ("Every AC has a completion note") checks. For `[MANUAL-VERIFY]`
items: note them as `✓ DONE — [MANUAL-VERIFY] — human to check in review`.

## Branch

Determine the branch in this order:

1. **Already on a non-main branch (anywhere in a stack):** use it as-is. This handles resumes and
   stacked PR workflows where the parent PR's branch already exists. Do not create a new branch.
2. **On main (or main equivalent) with a detectable in-progress branch for this task:** check it out
   and continue.
3. **On main with no matching branch:** derive `<slug>` from the task title (lowercase, hyphens, ≤6
   words) and create a new branch:

```bash
git checkout -b <branch-name>
```

Follow any repo-specific branch-naming convention you can infer from recent branches
(`git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short)' | head -20`).

## Implementation Loop

For each file in the implementation plan:

1. Read the existing file (if it exists) — do not assume its structure
2. Implement the change following the approved approach
3. Mark the workpad task list checkbox `[x]` when done
4. Commit logical units with descriptive messages

After each significant change, run the project's linter and tests — either the commands the project
documents or, if none are documented, the ones you can determine from its build files and toolchain.

If a test fails that was not touched: log it in the workpad under `### Pre-existing failures` as
`[PRE-EXISTING-FAILURE] file:line — description`. Do not fix it. Pre-existing failures do not block
completion bar item 3 — only new failures introduced by the current changes block it.

## Completion Bar

**Do not hand off to review until all 6 items are checked.**

```
COMPLETION BAR:
[ ] 1. All planned files created/modified (task list fully checked)
[ ] 2. Linter clean — zero new errors (pre-existing errors logged)
[ ] 3. No new test failures introduced — pre-existing failures logged as [PRE-EXISTING-FAILURE]
[ ] 4. Every AC has a completion note
[ ] 5. No open markers remain ([NEEDS CLARIFICATION], [TODO], [TODO-IMPL], [TBD])
[ ] 6. Scope discipline honored — improvements in [DEFERRED] blocks, never implemented
```

Work through any failing items before proceeding. Do not hand off with known gaps.

## Final Workpad Update

Before handing off to review, update workpad:

- Completion Bar with all 6 checked
- Branch name and latest sha
- Acceptance criteria status for each AC
- Any `[DEFERRED]` blocks discovered during implementation
- Phase Log: implement → done

Then push the branch (do not open a PR).

```bash
git push -u origin <branch-name>
```
