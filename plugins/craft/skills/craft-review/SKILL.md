---
name: craft-review
description: Use after craft-implement in the craft pipeline, or standalone to review any branch, PR, or diff. Selects reviewers from an inline topic map, spawns them in parallel, corroborates findings, and verifies acceptance criteria against the workpad when present.
argument-hint: "[branch, diff target, or workpad path]"
allowed-tools: Agent Bash Read Glob Grep WebFetch WebSearch
---

# /craft-review — Review

Arguments: $ARGUMENTS

---

## Role

Orchestrate a multi-angle review that proves the implementation satisfies its requirements.

Reviewer sub-agents are narrow by design. They are only useful when spawned together in this
orchestrated flow — invoking any individual reviewer directly produces a weak review.

## Interpreting Arguments

`$ARGUMENTS` can be any of the below. Resolve to a concrete `<base>` and `<head>` (or a set of paths
to restrict to), then **state your interpretation to the human in one line** before gathering the
diff.

- **Empty** → the current branch against its base.
- **Branch name** (e.g., `feature-x`) → that branch against its base. The base could be `main`, or
  it could be another feature branch in a stack.
- **Commit SHA or range** (`abc123`, `main..HEAD`, `abc123...def456`) → exactly that range.
- **PR reference** (`#42`, a PR URL) → `gh pr diff` + `gh pr view` for diff and metadata.
- **Workpad path** (e.g., `workpad.md`) → read the workpad for context; infer branch and base from
  it or the current checkout.
- **File or directory path** (`src/foo.ts`, `src/auth/`) → restrict the diff to that path, using the
  current branch's base as the diff base.
- **Natural-language description** → resolve to the most likely branch, PR, or diff target. Ask the
  human if ambiguous.

**Base detection** when reviewing a branch without an explicit base, review the current checkout vs
the current branch's base, including any dirty working changes.

**Interpretation line** (example): `Reviewing branch feature-x against base origin/main.`

## Step 1 — Gather Diff and Context

Write the resolved diff and changed-file list to temp files. Each sub-agent reads them from disk —
pasting the full diff into every reviewer's prompt would bloat context:

```bash
DIFF_FILE=$(mktemp /tmp/craft-review-diff.XXXXXX.patch)
FILES_FILE=$(mktemp /tmp/craft-review-files.XXXXXX.txt)
git diff <base>...<head> > "$DIFF_FILE"
git diff <base>...<head> --name-only > "$FILES_FILE"
```

Remember `$DIFF_FILE` and `$FILES_FILE` — you will pass their paths to every reviewer spawned in
Step 3.

Read the workpad (`workpad.md` or the path from the interpretation step, if any) for: requirements
R1..Rn, acceptance criteria, implementation summary. If the workpad is absent or has no
requirements, proceed without them but note in the Review Report: "No workpad found — requirements
traceability skipped."

## Step 2 — Reviewer Selection

Select reviewers by reading the diff against the map below.

**Always spawn (on every review):**

- `reviewer-correctness` — any logic change risks wrong outputs, broken contracts, invariant
  violations.
- `reviewer-simplification` — any code change risks over-engineering, dead code, unnecessary
  complexity.

**Conditional — spawn when the trigger matches:**

- `reviewer-requirements` — a workpad with requirements and/or acceptance criteria exists.
- `reviewer-idioms` — diff contains code in a detectable language or framework. Spawn one instance
  per distinct stack present (detect from file extensions and imports; pass the stack name in the
  prompt, e.g., "You are reviewing Go idioms.").
- `reviewer-security` — diff touches auth, authorization, input validation, secrets, injection,
  crypto, or permissions.
- `reviewer-memory-safety` — diff contains manual memory management (malloc/free, new/delete, unsafe
  blocks), pointer arithmetic, FFI/native interop, buffer operations, or code in languages with
  manual/semi-manual memory management (C, C++, Zig, Rust unsafe, CGo, JNI).
- `reviewer-performance` — diff touches hot paths, has O(n²) patterns, N+1 queries, allocations in
  loops, missing caching, or blocking in async contexts.
- `reviewer-tests` — test files added or modified, OR significant logic added without test changes.
- `reviewer-error-handling` — missing exception handlers, swallowed errors, silent failures, partial
  failure without cleanup, or retry/timeout gaps.
- `reviewer-api-surface` — public interfaces changed, exported types modified, new HTTP handlers,
  breaking API changes, or versioning.
- `reviewer-documentation` — missing docs on new public interfaces, stale comments contradicting
  changed code, or README sections invalidated by the diff.
- `reviewer-logging` — error paths with no log output, wrong log levels, unstructured logging in a
  structured-log codebase, or debug logging in prod paths.
- `reviewer-concurrency` (and `reviewer-correctness` in parallel) — shared mutable state without
  synchronization, lock ordering, data races, holding locks across awaits, or thread-unsafe
  collections.
- `reviewer-accessibility` — UI changes touching interactive elements, ARIA, form labels, keyboard
  navigation, focus management, or color-dependent information.
- `reviewer-responsiveness` — any new or changed UI layout in a web or native app, unless the app is
  clearly meant for a fixed viewport (game canvas, menu-bar app, kiosk, embedded widget).
- `reviewer-prose` — markdown, README sections, doc comments, error messages, UI strings,
  changelogs, or other human-facing prose.
- `reviewer-architecture` — new files or modules, new cross-module imports, dependency wiring
  changed, code moved between layers, or new interfaces or abstract types.

Select **all** relevant reviewers; there is no cap. Over-select rather than under-select. A false
positive adds one reviewer; a false negative misses a real issue.

## Step 3 — Parallel Review

Spawn all selected reviewers **in parallel** using the Agent tool with the matching `subagent_type`
(e.g., `"craft:reviewer-correctness"`). Each reviewer's prompt receives:

- The path to the diff file from Step 1 (`$DIFF_FILE`).
- The path to the changed-file list (`$FILES_FILE`).
- The path to the workpad, if available.
- For `reviewer-idioms`: the stack name (e.g., "The stack is: Go.")

## Step 4 — Cross-Review Corroboration

After all reviewers return, scan for the same finding across multiple outputs:

- **Match criteria:** same file:line (±3 lines) AND ≥80% text overlap, OR clearly the same root
  cause described differently.
- **Mark** corroborated findings with `[CORROBORATED-N]` where N is the number of reviewers who
  flagged it.
- **Effect on confidence:** corroborated LIKELY findings are treated as CERTAIN for downstream
  prioritization; use the signal to prioritize triage.
- **Deduplication:** show the corroborated finding once with the best-written description; note the
  other reviewer(s) in parentheses.

## Step 5 — Independent Verification of LIKELY findings

Collect all findings with confidence **LIKELY**. Batch them into a panel of **review-verifier**
instances (`subagent_type: "craft:review-verifier"`), grouping findings that share context (same
file, same library, same external API docs). For each finding, the verifier returns CONFIRMED, NOT
CONFIRMED, or REFUTED.

- **CONFIRMED:** finding stands at its stated severity.
- **NOT CONFIRMED:** verifier could not tell from inspection alone. Keep the finding and tag it
  `[UNVERIFIED]` in the review report so human triage weighs it accordingly.
- **REFUTED:** drop the finding.

Findings with confidence **CERTAIN** skip verification.

## Step 6 — Review Report

Produce the Review Report:

```markdown
## Review Report

Reviewers activated: [list]

### Findings

- **F-1** [P0] `auth.ts:88` — JWT not validated on refresh (correctness, human-triage)
  [CORROBORATED]
- **F-2** [P1] `utils.ts:12` — Null dereference on empty list (correctness, auto-fix)
- **F-3** [P1] `src/api.ts:—` — AC2 [R2] failed: grep for `X-Correlation-ID` not found
  (requirements, human-triage)
- **F-4** [P2] `api.ts:55` — Missing return type annotation (types, auto-fix)

### Reviewers with No Findings

[list reviewers that found nothing — e.g., `reviewer-requirements` listed here means every R was
covered and every AC passed its auto-verify]
```

**Finding Action classification** (reviewers decide this per finding):

- `auto-fix` — clear, unambiguous fix; no design decision required.
- `human-triage` — requires direction, judgment, or is legitimately deferrable.

## Workpad Update

If a workpad exists, write the review summary to `workpad.md` under `## Review`:

- Reviewers activated list
- Findings list

Update the Phase Log: review → done.

If no workpad exists (standalone invocation), skip this step — the review report in the conversation
is the deliverable.
