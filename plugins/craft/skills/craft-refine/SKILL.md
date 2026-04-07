---
name: craft-refine
description: Applies auto-fixes from review findings, batches remaining findings for human triage, then opens the PR. Requires review findings from a workpad or prior /craft-review run. HUMAN GATE before PR. Typically invoked by the /craft pipeline after review.
argument-hint: "[branch or workpad path]"
allowed-tools: Agent Bash Read Write Edit Glob Grep
---

# /craft-refine — Refine

Arguments: $ARGUMENTS

---

## Role

Turn the review into a mergeable PR.

## Resuming

If `$ARGUMENTS` is a workpad path, read the Review section to restore the findings list before
starting.

## Fast Path

If the review has zero findings (no auto-fix, no human-triage), skip directly to Step 5 (PR Feedback
Sweep). Output: "Review clean — no findings to address. Proceeding to PR."

## Step 1 — Apply Auto-fixes

Collect all findings marked `auto-fix` and apply them. For each:

- Make the change
- Commit: `fix: <finding title> (auto-fix from review)`

Run linters and tests after all auto-fixes. If something breaks, fix it before proceeding. Do not
proceed with a red linter or failing tests.

Output applied auto-fixes to the human and record them in the workpad:

```
Auto-fixed (N findings):
- F-2: Null dereference on empty list (utils.ts:12)
- F-3: Missing return type annotation (api.ts:55)
...
```

## Step 2 — Batch Human Triage

Collect all `human-triage` findings.

If there are zero human-triage findings, skip to Step 4 (second review round check). Output: "No
human-triage findings. Proceeding to PR."

Otherwise, present them in a single compact table, sorted by severity (P0 first):

```
Human triage required (N findings):

1. [P0] `auth.ts:88` — JWT not validated on refresh path (correctness)
2. [P1] `db.ts:14` — Missing transaction on multi-table write (correctness)
3. [P2] `api.ts:33` — Inconsistent error response shape (api-surface)
4. [P2] `auth.ts:102` — Token refresh doesn't rotate refresh token (security)
5. [P3] `utils.ts:7` — Function does two things, could be split (simplification)

Reply format: "fix 1-2, defer 3-4, reject 5" or per-item: "1=fix, 2=fix, 3=defer, 4=fix, 5=reject"
```

Output the following verbatim:

```markdown
## ⏸ HUMAN GATE — Review Triage

Auto-fixes above have been applied. For the remaining findings, reply with a triage decision for
each: **fix**, **defer**, or **reject**. Use bulk format — `"fix 1-2, defer 3-4, reject 5"` — or
per-item — `"1=fix, 2=defer, 3=reject"`.
```

## Step 3 — Apply Human-Triaged Fixes

After human responds, apply all `fix` decisions. For each fix: make the change, commit
`fix: <finding title>`.

Record `defer` decisions in workpad `## Deferred Items` section:

```
[DEFERRED]
title: [finding title]
severity: [P0|P1|P2|P3]
phase: review
context: [one-sentence finding summary]
[/DEFERRED]
```

Record `reject` decisions in workpad with reason if provided.

Run linters and tests after all fixes.

## Step 4 — Second Review Round (Conditional)

Trigger a second review round only if:

- A fix changed >50 lines of structural code, OR
- A fix added a new file, OR
- A fix touched a different module from the finding itself

If triggered: run `reviewer-correctness` and `reviewer-simplification` only, on the fix delta only
(not the full diff). Apply any new P0/P1 findings immediately — no second triage, as these are
regressions from the fixes themselves.

## Step 5 — PR Feedback Sweep

Before opening the PR, verify all deferred findings are recorded in the workpad. Check the workpad
for any lingering `[NEEDS CLARIFICATION]`, `[TODO]`, `[TODO-IMPL]`, or `[TBD]` markers. Resolve them
or add them to deferred findings. Do not open a PR with open markers.

## Step 6 — Open PR

Present summary for final approval:

Present the proposed PR title and a summary of what changed (auto-fixes applied, human-triaged
fixes, deferred findings). If the repo has a PR template (`.github/pull_request_template.md` or
similar), follow it. Otherwise use this default structure:

```
## Summary
[2-3 sentence description of what changed and why]

## Approach
[1 paragraph: chosen architecture and key decisions]

## Test plan
[How the changes were tested]
```

After approval, open a draft PR/MR using the project's code review tool (e.g.,
`gh pr create --draft` for GitHub). If the tool is unavailable or fails, output the title and body
so the human can open it manually. Record the URL (or failure reason) in the workpad.

## Step 7 — Wrap-up

After the PR is open, present a wrap-up prompt. If deferred findings exist, include them:

```markdown
## Wrap-up

**Deferred findings:** This task deferred [N] items.

- [P1] (review) Missing transaction on multi-table write
- [P2] (implement) Inconsistent error response shape
- ...

Would you like to file these as issues in your tracker? If so, specify the tool (e.g., GitHub
Issues, Linear, Jira) and any labels or project to apply.

**Workpad:** The workpad at `workpad.md` contains the full task history (requirements, architecture
decision, review findings, deferred items). Would you like to keep it, or clean it up?
```

If no deferred findings exist, skip the deferred section and only ask about the workpad.

Act on the human's response: file issues using the appropriate CLI tool (e.g., `gh issue create`),
clean up or preserve the workpad as requested.

## Rework Protocol

If the human requests significant rework (structural change or new requirement):

1. If a PR/MR is open: close it
2. Create new branch: `<branch-name>-v2` (increment version suffix each time)
3. Read the old `workpad.md` and copy out the Requirements section (R1..Rn plus any new requirements
   from the rework instruction)
4. Rename old workpad: `mv workpad.md workpad-v1.md`
5. Create fresh `workpad.md` from `${CLAUDE_PLUGIN_ROOT}/references/workpad-template.md`
6. Write the copied requirements into `## Requirements` in the new workpad
7. Write the rework instruction into `## Task` as an amendment
8. Return to `/craft-architect`, not `/craft-implement`

Rework is a reset, not a patch. The old workpad is preserved as `workpad-v1.md` for reference.
