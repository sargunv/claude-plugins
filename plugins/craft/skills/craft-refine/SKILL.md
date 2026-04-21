---
name: craft-refine
description: Use after craft-review in the craft pipeline. Applies auto-fixes from review findings, batches remaining findings for human triage, and re-runs a narrow review on fix deltas when warranted. Leaves the branch with fixes applied and deferred items recorded in the workpad. HUMAN GATE for triage.
argument-hint: "[branch or workpad path]"
allowed-tools: Agent Bash Read Write Edit Glob Grep
---

# /craft-refine — Refine

Arguments: $ARGUMENTS

---

## Role

Turn review findings into clean code. Apply the auto-fixes, triage the rest with the human, and
leave the branch ready for PR.

## Resuming

If `$ARGUMENTS` is a workpad path, read the Review section to restore the findings list before
starting.

## Fast Path

If the review has zero findings (no auto-fix, no human-triage), skip directly to Step 5 (feedback
sweep). Output: "Review clean — no findings to address."

## Step 1 — Apply Auto-fixes

Collect all findings marked `auto-fix` and apply them. For each:

- Make the change.
- Commit: `fix: <finding title> (auto-fix from review)`.

Run the project's linters and tests after all auto-fixes — either the commands the project documents
or, if none are documented, the ones you can determine from its build files and toolchain. If
something breaks, fix it before proceeding. Do not proceed with a red linter or failing tests.

Output applied auto-fixes and record them in the workpad:

```
Auto-fixed (N findings):
- F-2: Null dereference on empty list (utils.ts:12)
- F-3: Missing return type annotation (api.ts:55)
...
```

## Step 2 — Batch Human Triage

Collect all `human-triage` findings.

If there are zero human-triage findings, skip to Step 4 (second review round check). Output: "No
human-triage findings."

Otherwise, present them in one or more compact lists, grouped by domain and sorted by severity (P0
first):

```
Human triage required (N findings):

1. [P0] `auth.ts:88` — JWT not validated on refresh path (correctness)
2. [P1] `db.ts:14` — Missing transaction on multi-table write (correctness)
3. [P2] `api.ts:33` — Inconsistent error response shape (api-surface)
4. [P2] `auth.ts:102` — Token refresh doesn't rotate refresh token (security)
5. [P3] `utils.ts:7` — Function does two things, could be split (simplification)
```

Output the following verbatim:

```markdown
## ⏸ HUMAN GATE — Review Triage

Auto-fixes above have been applied. For the remaining findings, reply with a triage decision for
each: **fix**, **defer**, or **reject**. Use bulk format — `"fix 1-2, defer 3-4, reject 5"` — or
per-item — `"1=fix, 2=defer, 3=reject"`.
```

## Step 3 — Apply Human-Triaged Fixes

After the human responds, apply all `fix` decisions. For each fix: make the change, commit
`fix: <finding title>`.

Record `defer` decisions in workpad `## Deferred Items` section:

```
- [finding title]
  - severity: [P0|P1|P2|P3]
  - phase: review
  - context: [one-sentence finding summary]
```

Record `reject` decisions in workpad with reason if provided.

Run linters and tests after all fixes.

## Step 4 — Second Review Round (Conditional)

Trigger a second review round if:

- A fix changed structural code, OR
- A fix added a new file, OR
- A fix touched a different module from the finding itself.

If triggered, write the fix delta to a temp file and pass its path to `reviewer-correctness` and
`reviewer-simplification`:

```bash
DELTA_FILE=$(mktemp /tmp/craft-refine-delta.XXXXXX.patch)
git diff <pre-fix-sha>..HEAD > "$DELTA_FILE"
```

Apply any new P0/P1 findings immediately — no second triage, as these are regressions from the fixes
themselves.

## Step 5 — Feedback Sweep

Verify all deferred findings are recorded in the workpad. Check the workpad for any lingering
`[NEEDS CLARIFICATION]`, `[TODO]`, `[TODO-IMPL]`, or `[TBD]` markers. Resolve them or add them to
deferred findings. Do not hand off with open markers.

Update the Phase Log: refine → done.
