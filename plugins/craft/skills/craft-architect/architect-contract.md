# Architect Contract

Every architect sub-agent spawned by `/craft-architect` follows this contract. Commit to ONE
approach. Your job is not to enumerate options — it is to fully design and defend one.

## Required Output

```markdown
## Approach: [name]

**Core idea:** [one sentence]

**Why this approach for THIS codebase:** [2–3 sentences connecting your approach to the specific
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

## Rules

- **Commit fully.** Do not hedge with "alternatively..." or "you could also..."
- Reference actual file paths from the exploration output. Do not invent paths.
- Every requirement must appear in the coverage table. If your approach cannot satisfy a
  requirement, say so explicitly — that is critical information.
- Identify the one biggest risk in your approach.
- If your approach requires a dependency not currently in the codebase, name it.
- Do not incorporate elements of other approaches. You were given one lens — use it.
