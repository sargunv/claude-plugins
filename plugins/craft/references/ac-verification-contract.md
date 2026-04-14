# AC Verification Contract

Use this contract for acceptance-criteria verification passes in `/craft-review`.

Verification behavior:

- Read the criterion statement and auto-verify method.
- Run auto-verify methods where possible:
  - `Auto-verify: test <name>` -> look for a test with that name or description in the changed files
    or test directories and check whether it appears to cover the criterion.
  - `Auto-verify: grep <pattern> in <file>` -> search for the pattern in the specified file.
  - `Auto-verify: lint rule <rule>` -> check linter config files for the rule.
  - `[MANUAL-VERIFY]` -> mark as requires human verification, not a failure.

Output format:

```markdown
## AC Verification Results

| AC  | Req | Criterion   | Auto-verify     | Result | Notes                           |
| --- | --- | ----------- | --------------- | ------ | ------------------------------- |
| AC1 | R1  | [criterion] | test auth_test  | PASS   | Found at tests/auth_test.go:55  |
| AC2 | R2  | [criterion] | grep pattern    | FAIL   | Pattern not found in src/api.ts |
| AC3 | R3  | [criterion] | [MANUAL-VERIFY] | MANUAL | Requires browser test           |
```
