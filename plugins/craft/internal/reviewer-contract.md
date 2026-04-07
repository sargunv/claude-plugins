# Reviewer Contract

Every reviewer agent must implement this contract. The review orchestrator depends on consistent
section names, finding format, and severity definitions across all reviewers.

## Standard Finding Format

All reviewers use this format. Every field is required.

```markdown
### F-[N]: [concise title]

**File:** `path/to/file:line` **Severity:** [see severity definitions below] **Action:** auto-fix |
human-triage **Confidence:** CERTAIN | LIKELY | POSSIBLE **Confidence basis:** [one sentence
explaining the confidence rating]

**Finding:** [what is wrong — describe the problem, not the fix]

**Evidence:**
```

[exact code excerpt from the diff]

```
**Fix:** [specific and concrete — a code snippet if the fix is ≤8 lines, prose if longer]
```

Every field must be present in every finding. The review orchestrator parses the finding table from
these fields; a missing field causes the finding to be dropped. The corroboration pass and
independent verification pass also depend on consistent file:line references.

---

## Severity Definitions

| Severity        | Meaning                                                                                                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **P0-BLOCKER**  | Defect that causes data loss, a security breach, or a crash in production on a code path that will be exercised. Must be fixed before the PR merges.                          |
| **P1-HIGH**     | Defect that produces incorrect behavior or a significant user-facing failure in a realistic scenario. Should be fixed before merge; deferral requires explicit justification. |
| **P2-MEDIUM**   | Issue that degrades quality — readability, maintainability, or minor correctness — without causing an immediate user-facing failure. Fix in this PR or file for the next.     |
| **P3-LOW**      | Minor issue: style, naming, small simplification opportunity, or advisory note. Fix at discretion.                                                                            |
| **P4-ADVISORY** | Observation with no required action. Use for noting patterns, flagging potential future issues, or recording context.                                                         |

---

## Action Classification

`auto-fix` — the fix is clear, unambiguous, and requires no design judgment. The refine phase
applies these without human input. Use `auto-fix` when the correct code can be written from the
finding description alone, with no tradeoffs to weigh.

`human-triage` — the fix requires a design decision, involves a tradeoff, or is legitimately
deferrable. The refine phase presents these to the human in a batch table. Use `human-triage` for
anything where the right answer is not obvious, where fixing it might affect other parts of the
system, or where the human may reasonably choose to defer or reject the finding.

When the classification is uncertain, use `human-triage`. Misclassifying a `human-triage` finding as
`auto-fix` causes the refine phase to make a design decision without human input.

---

## Confidence Threshold

Publish findings at LIKELY or CERTAIN confidence. Do not publish POSSIBLE findings unless you are
the security reviewer. The review orchestrator automatically spawns independent verification for P0
and P1 findings with POSSIBLE confidence; a steady stream of unverified POSSIBLE findings slows the
pipeline.

Exception: `reviewer-security` may publish POSSIBLE findings at P0 and P1 severity. Security
findings have asymmetric cost — missing a real vulnerability is worse than a false positive. When
publishing a POSSIBLE security finding, explain clearly why the concern might not materialize.

---

## Confidence Calibration (Generic Definitions)

**CERTAIN** — You can point to the exact file and line where the issue exists, describe the precise
input or condition that triggers it, and trace the failure to a concrete outcome. No inferential
steps are required.

**LIKELY** — Strong evidence the issue exists. You have traced the relevant code path and identified
the probable failure condition, but have not fully verified that the trigger condition is reachable
or that the outcome is as described.

**POSSIBLE** — Real evidence of a concern, but the issue may not materialize depending on context
you cannot verify by reading the diff alone. The fix would be correct if the concern is valid, but a
false positive is plausible.

---

## Out of Scope

When a reviewer notices an issue that falls in another reviewer's domain, record it in a section
named `## Out of Scope` at the end of the output. Format: one line per observation — file:line and
one sentence describing the concern. Do not analyze it. Do not assign severity. The review
orchestrator uses this section to identify coverage gaps and, if needed, request targeted re-review.
