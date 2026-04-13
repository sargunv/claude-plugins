---
name: craft-review
description: This skill should be used when the user asks to "review my code", "check this branch", "what's wrong with my changes", "review the PR", or wants a code review on any branch, PR, or diff. Multi-reviewer code review that topic-tags the diff, spawns targeted reviewers in parallel, verifies acceptance criteria, and scores findings.
argument-hint: "[branch, diff target, or workpad path]"
allowed-tools: Agent Bash Read Glob Grep WebFetch WebSearch
---

# /craft-review — Review

Arguments: $ARGUMENTS

---

## Supporting files

- [topic-reviewer-map.md](topic-reviewer-map.md) — maps topic tags to reviewer agents; read during
  reviewer selection

## Role

Orchestrate a multi-angle review that proves the implementation satisfies its requirements.

## Resuming

If `$ARGUMENTS` is a branch name or diff target, review that branch. If a workpad exists, use it for
requirements traceability and AC verification. If not, proceed without it and note the gap.

## Step 0 — Preflight Check

Before starting: read the workpad and check the Completion Bar section. If any of the 6 items are
unchecked, stop and report: "Implementation completion bar is incomplete — items [N, N] not checked.
Review cannot proceed until implementation is verified complete." (Exception: if `/craft-review` is
called directly on an ad-hoc branch without a workpad, skip this check and note "No workpad —
completion bar check skipped.")

## Step 1 — Gather Diff and Context

Save the diff and changed-file list to temporary files so that sub-agents can read them directly
(avoids copying the full diff into every agent prompt). Use `mktemp -d` to create a unique directory
so concurrent reviews across projects or worktrees don't collide:

```bash
REVIEW_TMPDIR=$(mktemp -d)
(git diff ${ARGUMENTS:-main}...HEAD 2>/dev/null || git diff HEAD~1) > "$REVIEW_TMPDIR/diff.patch"
(git diff ${ARGUMENTS:-main}...HEAD --name-only 2>/dev/null || git diff HEAD~1 --name-only) > "$REVIEW_TMPDIR/files.txt"
```

Read the workpad (`workpad.md`) for: requirements R1..Rn, acceptance criteria, implementation
summary. If workpad is absent or has no requirements, proceed without them but note in the Review
Report: "No workpad found — requirements traceability and AC verification skipped."

## Step 2 — Review

Read the topic-reviewer map. Each reviewer agent has a matching `subagent_type` (e.g.,
`"craft:reviewer-correctness"`, `"craft:reviewer-simplification"`, etc.).

Tell every reviewer to collect context themselves. Pass every reviewer:

- The path to the diff file: `$REVIEW_TMPDIR/diff.patch` (agent reads it via the Read tool)
- The path to the changed-file list: `$REVIEW_TMPDIR/files.txt` (agent reads it, then reads each
  listed file to get the full content)
- Requirements R1..Rn from workpad (≤300 words), if available
- Implementation summary from workpad (≤200 words), if available

Do NOT paste the diff or file contents into the agent prompt — the agents will read them directly.

Each reviewer returns structured findings using the finding format defined in their agent file.

### Wave 1 — always-on reviewers + topic tagger

Spawn the following **in parallel**, in a single message:

- All reviewers marked `✓` in the Always? column of the map (`reviewer-correctness`,
  `reviewer-simplification`, `reviewer-requirements`)
- The **topic-tagger** sub-agent (`subagent_type: "craft:topic-tagger"`), passing it:
  - The path to the diff file: `$REVIEW_TMPDIR/diff.patch`
  - The path to the changed-file list: `$REVIEW_TMPDIR/files.txt`
  - The tag vocabulary from the topic-reviewer map

The tagger emits conditional topic tags only (not the always-on ones). Reviewer selection happens in
wave 2.

### Wave 2 — conditional reviewers

After the topic tagger returns, select and spawn conditional reviewers:

- For each tag emitted by the tagger, spawn the reviewer(s) listed in the topic-reviewer map's
  Reviewer(s) column.
- For `reviewer-idioms`: spawn one instance per `idioms-*` tag. Pass the stack name as a parameter.
  Example: tag `idioms-go` → spawn reviewer-idioms with "Review Go idioms. The stack is: Go."
- **Coverage floor:** If the tagger's tags select zero conditional reviewers and the diff exceeds 50
  lines, add `reviewer-tests` and `reviewer-error-handling` as defaults. Log: "Topic tagger emitted
  no conditional tags for >50 line diff. Adding tests + error-handling as coverage floor."
- Deduplicate against wave 1 reviewers before spawning.

## Step 3 — Cross-Review Corroboration

After all reviewers return, scan for the same finding across multiple outputs:

- **Match criteria:** Same file:line (±3 lines) AND ≥80% text overlap, OR clearly the same root
  cause described differently
- **Mark** corroborated findings with `[CORROBORATED-N]` where N is the number of reviewers who
  flagged it
- **Effect on confidence:** Corroborated POSSIBLE findings upgrade to LIKELY; LIKELY remains LIKELY
  (no auto-severity change, but use this signal to prioritize triage)
- **Deduplication:** Show the corroborated finding once with the best-written description; note the
  other reviewer(s) in parentheses

## Step 4 — AC Verification

Skip this step if no workpad or acceptance criteria exist. Note in the Review Report: "No acceptance
criteria — AC verification skipped."

Spawn the **review-verifier** sub-agent using the Agent tool with
`subagent_type: "craft:review-verifier"`.

Pass to it:

- The acceptance criteria from workpad
- The path to the changed-file list: `$REVIEW_TMPDIR/files.txt` (agent reads each listed file)

The verifier checks each AC against the implementation:

- Runs auto-verify methods where possible
- Returns PASS / FAIL / MANUAL-VERIFY for each AC
- FAIL becomes a P1 finding in the review report

## Step 5 — Independent Verification

Collect all findings with confidence **POSSIBLE or LIKELY**. Batch them into a reasonable number of
**review-verifier** instances (using `subagent_type: "craft:review-verifier"`), grouping findings
that share context (e.g., same file, same library, same external API docs). For each finding, the
verifier returns a binary verdict: CONFIRMED / NOT CONFIRMED.

- If CONFIRMED: finding stands at its stated severity.
- If NOT CONFIRMED: drop the finding.

Findings with confidence **CERTAIN** skip verification.

## Step 6 — Scoring and Synthesis

Compute review score (informational — not a hard gate):

- Start at 100
- P0 finding: −25 (floor at 0)
- P1 finding: −8
- P2 finding: −3
- P3 finding: −1
- Unaddressed requirement: −15 each Note: failed ACs are scored as P1 findings (−8 each) via
  review-verifier output. Do not apply a separate Failed AC penalty — it would double-count.

Produce the **Review Report**:

```markdown
## Review Report

Score: [0–100] ([pass/needs-attention]) Reviewers activated: [list]

### Findings

- **F-1** [P0] `auth.ts:88` — JWT not validated on refresh (correctness, medium, human-triage)
  [CORROBORATED]
- **F-2** [P1] `utils.ts:12` — Null dereference on empty list (correctness, trivial, auto-fix)
- **F-3** [P2] `api.ts:55` — Missing return type annotation (types, trivial, auto-fix)

### AC Verification

- AC1 [R1]: PASS — test auth_test.ts
- AC2 [R2]: FAIL — grep pattern not found

### Reviewers with No Findings

[list reviewers that found nothing above confidence threshold]
```

**Finding Action classification** (reviewers decide this per finding):

- `auto-fix` — clear, unambiguous fix; no design decision required
- `human-triage` — requires direction, judgment, or is legitimately deferrable

## Workpad Update

If a workpad exists, write the review summary to `workpad.md` under `## Review`:

- Score
- Reviewers activated list
- Findings list
- AC Verification list

Then update the Phase Log: review → done, with timestamp and score.

If no workpad exists (standalone invocation), skip this step — the review report in the conversation
is the deliverable. If the review produced findings, append:

> Run `/craft-refine` to apply auto-fixes and triage the remaining findings.
