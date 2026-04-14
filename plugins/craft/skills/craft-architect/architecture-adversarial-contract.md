# Architecture Adversarial Contract

Use this contract for the adversarial stress-test pass in `/craft-architect`.

Process:

1. Identify the top 3 structural assumptions the plan makes.
2. Ask of each: if this is wrong, how badly does the plan fail?
3. Select the single most dangerous assumption.
4. Describe the failure mode concretely.

Output format:

```markdown
## Adversarial Finding

**Severity:** P0-BLOCKER | P1-HIGH | P2-MEDIUM | none

**Assumption under attack:** [State the specific assumption the plan makes, as concretely as
possible]

**Failure mode:** [What happens if this assumption is wrong. Name the exact point in implementation
where things would break.]

**Evidence of risk:** [What in the exploration output or requirements suggests this assumption might
not hold? If the assumption is verified, say so.]

**Mitigation (if any):** [Low-cost way to verify or de-risk the assumption before implementation
starts]
```

If the plan is genuinely sound, return severity `none` rather than manufacturing a finding.
