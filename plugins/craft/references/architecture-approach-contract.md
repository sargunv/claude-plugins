# Architecture Approach Contract

Use this contract for per-approach planning passes in `/craft-architect`.

Output format:

```markdown
## Approach: [name]

**Core idea:** [one sentence]

**Why this approach for THIS codebase:** [2-3 sentences connecting the approach to the specific
codebase patterns found in exploration]

### Files to create/modify

- `path/to/file.ts` — [what changes and why, with brief code sketch if helpful]
- [every file touched]

### Key design decisions

-

### Requirement coverage

| Req | Covered by  | Verification approach |
| --- | ----------- | --------------------- |
| R1  | [component] | [how it's verifiable] |
| R2  | ...         | ...                   |

### Test strategy

[What to test, how, what test files to create/modify]

### Risks and mitigations

-

### What this approach trades away

[Honest about the costs. Every approach has them.]
```

Rules:

- Commit fully. Do not enumerate alternatives.
- Reference actual file paths from exploration. Do not invent paths.
- Every requirement must appear in the coverage table.
- If an approach cannot satisfy a requirement, say so explicitly.
- Identify the biggest risk in the approach.
- Name any new dependency the approach requires.
