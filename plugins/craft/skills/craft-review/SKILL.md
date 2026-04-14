---
name: craft-review
description: This skill should be used when the user asks to "review my code", "check this branch", "what's wrong with my changes", "review the PR", or wants a code review on any branch, PR, or diff. Multi-reviewer code review that runs required reviews, selects conditional reviews from the diff, verifies acceptance criteria, and scores findings.
argument-hint: "[branch, diff target, or workpad path]"
---

# /craft-review — Review

Arguments: $ARGUMENTS

---

## Supporting files

- [reviewer-contract.md](../../references/reviewer-contract.md) — every reviewer sub-agent must read
  this before reviewing and must follow its finding format exactly

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

Tell every reviewer to collect context themselves. Pass every reviewer:

- The path to the diff file: `$REVIEW_TMPDIR/diff.patch` (agent reads it via the Read tool)
- The path to the changed-file list: `$REVIEW_TMPDIR/files.txt` (agent reads it, then reads each
  listed file to get the full content)
- The path to the shared reviewer contract: `../../references/reviewer-contract.md` relative to this
  skill file (agent reads it before reviewing)
- Requirements R1..Rn from workpad (≤300 words), if available
- Implementation summary from workpad (≤200 words), if available

Do NOT paste the diff or file contents into the agent prompt — the agents will read them directly.

Each reviewer returns structured findings using the format from `reviewer-contract.md`.

### Required reviews

Spawn these reviewers **in parallel** for every review:

- `correctness` — always run; focus on broken behavior, wrong outputs, violated contracts, and
  invariant failures.
- `simplification` — always run; focus on unnecessary complexity, over-engineering, dead code, and
  opportunities to achieve the same outcome with a smaller or clearer change.
- `requirements` — run when a workpad with R1..Rn exists; focus on whether the implementation
  actually satisfies the stated requirements and acceptance criteria.

### Conditional reviews

Select conditional reviewers by inspecting the diff and changed-file list directly. Spawn all that
apply, in parallel:

- `idioms` — run once per distinct language or framework in the changed files; focus on stack-
  specific conventions and misuse of common patterns. If multiple stacks appear in the diff, spawn
  one idioms reviewer per stack and tell each one which stack it owns.
- `security` — run when the diff touches auth, authorization, input validation, secrets handling,
  cryptography, permissions, or other security-sensitive boundaries; focus on vulnerabilities and
  exposure risks.
- `performance` — run when the diff touches hot paths, large loops, database/query paths, caching,
  allocation-heavy code, or blocking operations; focus on throughput, latency, and avoidable work.
- `tests` — run when test files changed, significant logic changed, or new logic was added without
  corresponding tests; focus on missing coverage, weak assertions, and misleading tests.
- `error-handling` — run when the diff introduces new failure paths, retries, timeouts, external
  calls, parsing, I/O, or partial state changes; focus on missing handlers, swallowed errors, and
  cleanup gaps.
- `api-surface` — run when exported types, public functions, handlers, schemas, or wire contracts
  changed; focus on breaking changes, naming, versioning, and external compatibility.
- `documentation` — run when public interfaces changed, comments changed, or user-facing docs may
  now be stale; focus on missing or misleading documentation.
- `logging` — run when operational code paths, especially error paths or integrations, changed;
  focus on missing logs, wrong log levels, and broken structured logging.
- `concurrency` — run when shared state, async coordination, locks, queues, workers, or thread-like
  behavior changed; focus on races, ordering bugs, and unsafe cross-task interactions.
- `accessibility` — run when UI code changes touch interactive elements, forms, focus management,
  keyboard handling, ARIA, or color-dependent information; focus on user accessibility regressions.
- `memory-safety` — run when the diff touches manual memory management, unsafe blocks, FFI, native
  interop, pointers, or buffer manipulation; focus on lifetime, aliasing, bounds, and ownership
  errors.
- `prose` — run when markdown, README sections, doc comments, error messages, UI copy, changelogs,
  or other human-facing text changed; focus on clarity, correctness, and contradictions with the
  code.
- `architecture` — run when new modules, new files, new cross-module imports, dependency wiring, or
  layer boundaries changed; focus on layering, coupling, and fit with the existing structure.

Coverage floor: if no conditional review applies and the diff exceeds 50 lines, add `tests` and
`error-handling` anyway.

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

Spawn one verification pass for acceptance criteria.

Pass to it:

- The acceptance criteria from workpad
- The path to the changed-file list: `$REVIEW_TMPDIR/files.txt` (agent reads each listed file)

The verifier checks each AC against the implementation:

- Reads the criterion statement and auto-verify method
- Runs auto-verify methods where possible:
  - `Auto-verify: test <name>` -> look for a test with that name or description in the changed files
    or test directories and check whether it appears to cover the criterion
  - `Auto-verify: grep <pattern> in <file>` -> search for the pattern in the specified file
  - `Auto-verify: lint rule <rule>` -> check linter config files for the rule
  - `[MANUAL-VERIFY]` -> mark as requires human verification, not a failure
- Returns PASS / FAIL / MANUAL-VERIFY for each AC using this format:

```markdown
## AC Verification Results

| AC  | Req | Criterion   | Auto-verify     | Result | Notes                           |
| --- | --- | ----------- | --------------- | ------ | ------------------------------- |
| AC1 | R1  | [criterion] | test auth_test  | PASS   | Found at tests/auth_test.go:55  |
| AC2 | R2  | [criterion] | grep pattern    | FAIL   | Pattern not found in src/api.ts |
| AC3 | R3  | [criterion] | [MANUAL-VERIFY] | MANUAL | Requires browser test           |
```

- FAIL becomes a P1 finding in the review report

## Step 5 — Independent Verification

Collect all findings with confidence **POSSIBLE or LIKELY**. Batch them into a reasonable number of
verification passes, grouping findings that share context (e.g., same file, same library, same
external API docs). For each finding, the verifier returns a binary verdict: CONFIRMED / NOT
CONFIRMED.

For each finding, the verification pass should:

- Read the finding description and evidence
- Read the specified file at the specified line
- Trace the code path if needed
- Return one of two verdicts only, using this format:

```markdown
## Verdict: [CONFIRMED | NOT CONFIRMED] — [finding title]

**File:** `path/to/file:line`

**Reasoning:** [one paragraph — what you found when you examined the code]
```

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
- Unaddressed requirement: −15 each Note: failed ACs are scored as P1 findings (−8 each) via the AC
  verification output. Do not apply a separate Failed AC penalty — it would double-count.

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
