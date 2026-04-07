# Topic → Reviewer Map

This file is the single source of truth for:

1. What topic tags exist (the Tag column)
2. What each tag means (so the topic-tagger knows what to emit)
3. Which reviewer agents activate for each tag

The topic-tagger reads the Description column to understand what to emit. The review orchestrator
reads the Reviewer(s) column to decide which agents to launch.

## Adding a reviewer

1. Create `agents/reviewer-<name>.md` following the contract in
   `${CLAUDE_PLUGIN_ROOT}/internal/reviewer-contract.md`
2. Add a row to this table, or add the reviewer to an existing row
3. Done. No other files need changing.

## Adding a new tag

1. Add a row with a clear description
2. List the reviewer(s) to activate
3. The topic-tagger will pick it up automatically on next run

---

## Tag Table

| Tag              | Description (topic-tagger reads this)                                                                                                                                        | Always?     | Reviewer(s)                                    |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------------------------------------------- |
| `correctness`    | Any logic change — wrong outputs, broken contracts, invariant violations                                                                                                     | ✓           | `reviewer-correctness`                         |
| `simplification` | Any code change — over-engineering, dead code, unnecessary complexity                                                                                                        | ✓           | `reviewer-simplification`                      |
| `requirements`   | Always checked — verifies R1..Rn from workpad are addressed                                                                                                                  | ✓           | `reviewer-requirements`                        |
| `idioms`         | Language/framework conventions — emit as `idioms-<lang>` or `idioms-<framework>` (e.g. `idioms-go`, `idioms-react`). Emit one tag per distinct stack found in changed files. | ✓ per stack | `reviewer-idioms` (parameterized by stack)     |
| `security`       | Auth, authorization, input validation, secrets exposure, injection, cryptography, permissions                                                                                |             | `reviewer-security`                            |
| `performance`    | Hot paths, O(n²) patterns, N+1 queries, unnecessary allocations, missing caching, blocking operations                                                                        |             | `reviewer-performance`                         |
| `tests`          | Test files added/modified; significant logic added without test changes; test quality or coverage gaps                                                                       |             | `reviewer-tests`                               |
| `error-handling` | Missing exception handlers, swallowed errors, silent failures, partial failure without cleanup, retry logic, timeouts                                                        |             | `reviewer-error-handling`                      |
| `api-surface`    | Public interfaces changed, exported types modified, breaking changes, API naming, versioning                                                                                 |             | `reviewer-api-surface`                         |
| `documentation`  | Missing docs on new public interfaces, stale comments that contradict changed code, misleading parameter/return docs, README sections invalidated by the diff                |             | `reviewer-documentation`                       |
| `logging`        | Error paths with no log output, wrong log levels, unstructured logging in structured-logging codebases, debug logging left in production paths                               |             | `reviewer-logging`                             |
| `concurrency`    | Shared mutable state without synchronization, lock-ordering violations, data races, holding locks across await points, thread-unsafe collection usage                        |             | `reviewer-concurrency`, `reviewer-correctness` |
| `accessibility`  | UI changes touching interactive elements, ARIA attributes, form labels, keyboard navigation, focus management, or color-dependent information                                |             | `reviewer-accessibility`                       |
| `memory-safety`  | Manual memory management (malloc/free, new/delete, unsafe blocks), pointer arithmetic, FFI/native interop, buffer operations                                                 |             | `reviewer-memory-safety`                       |
| `prose`          | Markdown files, README sections, doc comments, error messages, UI strings, changelogs, or other human-facing prose added or modified in the diff                             |             | `reviewer-prose`                               |
| `architecture`   | New files or modules added, new cross-module imports introduced, dependency wiring changed, code moved between layers or packages, new interfaces or abstract types defined  |             | `reviewer-architecture`                        |

---

## Notes

- The first three rows (`correctness`, `simplification`, `requirements`) are always-on. The
  topic-tagger should NOT emit these (the orchestrator adds them automatically).
- For `idioms`, the tagger emits `idioms-typescript`, `idioms-go`, `idioms-python`, etc. based on
  file extensions in the diff. One tag per detected stack. The orchestrator spawns one
  `reviewer-idioms` instance per idioms tag, passing the stack name as a parameter.
- If tagger emits zero conditional tags for a diff >50 lines, the orchestrator defaults to adding
  `tests` and `error-handling` as a coverage floor.
