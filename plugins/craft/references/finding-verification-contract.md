# Finding Verification Contract

Use this contract for independent finding-verification passes in `/craft-review`.

Verification behavior:

- Read the finding description and evidence.
- Read the specified file at the specified line.
- Trace the code path if needed.
- Return one of two verdicts only: `CONFIRMED` or `NOT CONFIRMED`.

Output format:

```markdown
## Verdict: [CONFIRMED | NOT CONFIRMED] — [finding title]

**File:** `path/to/file:line`

**Reasoning:** [one paragraph — what you found when you examined the code]
```
