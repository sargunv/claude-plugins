# Explorer Contract

Every explorer sub-agent spawned by `/craft-explore` follows this contract. The orchestrator depends
on consistent section names and output format across agents to synthesize the exploration report.

## Required Output

Your output has exactly four sections.

### Key Facts (≤20 bullets)

Dense signal. Lead with the most actionable facts. Every bullet that references code must include
the exact file path (and line number when relevant).

- Specific, not vague. "auth middleware in `src/auth/middleware.ts:47` uses JWT with 1-hour expiry"
  ✓
- Never: "the codebase uses TypeScript" ✗ (too vague)
- Facts about what is ABSENT are as valuable as facts about what is PRESENT.

### Code Snippets (required)

Exact snippets from the codebase — never pseudocode. For each significant pattern:

```
// FILE: path/to/file.ts:42-55
[exact code excerpt]
```

Label each snippet with why it matters for the task.

### Key Files

List the 5–10 most essential files for understanding this focus area. Exact path and one-sentence
description per file.

### Appendix

Lower-priority observations. Full details. Less filtered.

## Rules

- Read actual files before making claims. Do not speculate about what might be there.
- Note what you did not find (expected files/patterns that are absent).
- If something is ambiguous, say so explicitly — do not guess.
- Never make implementation recommendations. Your job is facts, not solutions.
- Stay within the focus area you were given. Note out-of-scope observations briefly but do not
  analyze them.
- If an external library or API is relevant and its behavior is not clear from local files, fetch
  its documentation before reporting on it.
