---
name: craft-architect
description: Use after craft-clarify in the craft pipeline. Designs an implementation plan, stress-tests it with an adversarial sub-agent, then presents a single recommendation. Requires requirements (R1..Rn) from a workpad or inline context. HUMAN GATE.
argument-hint: "[workpad path or inline context]"
allowed-tools: Agent Read Glob Grep WebFetch WebSearch
---

# /craft-architect — Architecture

Arguments: $ARGUMENTS

---

## Role

Run thorough architectural analysis before a single line of code is written.

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context (task description, exploration report,
requirements R1..Rn). The human gate fires again — that is expected.

## Setup

Read the workpad (`workpad.md` or `$ARGUMENTS`) to retrieve:

- Task description
- Exploration Report (Key Facts + Key Files + code snippets)
- Requirements R1..Rn from clarify

If the workpad is missing or does not contain requirements R1..Rn, stop and ask for it. **Do not
proceed without requirements — architecture without requirements produces plans that fail review.**

Read the key files surfaced during exploration before designing.

## Architect Agents

Spawn architect sub-agents **in parallel** using the Agent tool. Each commits fully to ONE approach
— no hedging, no combining. The orchestrator picks the best (or deliberately combines) after they
all return. Forcing commitment in separate contexts prevents the middle-ground-hedge pattern a
single-context pass falls into.

**Always spawn (floor):**

- **Minimal** — fewest moving parts, smallest diff, maximum reuse of existing code.
- **Cleanest/Robust** — correct abstractions, full error handling, future-proof design.

**Add for complex tasks** (new subsystem, API surface change, data model change, cross-cutting work
across many modules):

- **Pragmatic** — best fit for this codebase's actual patterns, fastest to implement correctly.

**Add when relevant:**

- **Performance** — optimize for throughput/latency. Spawn when performance is a stated requirement.
- **Incremental** — staged rollout with feature flags. Spawn when the change is high-risk or
  zero-downtime deployment is required.

## Invocation Prompt for Each Agent

Pass each sub-agent:

- Task description
- Exploration key facts + relevant code snippets (from the workpad)
- Requirements R1..Rn
- Its assigned approach directive (verbatim from above)
- An instruction to read `${CLAUDE_PLUGIN_ROOT}/skills/craft-architect/architect-contract.md` for
  the required output format and rules

## Synthesis

After all architect agents return:

1. **Per-agent summaries** — distill each proposal into a tweet-sized summary capturing what it does
   and how. Include an additional tweet-sized summary of dimensions where all proposals agree.
2. **Tradeoff pass** — for each dimension where proposals meaningfully differ (scope, complexity,
   risk, requirement coverage), state the tradeoff as prose. Omit dimensions where all proposals
   agree.
3. **Commit** — pick the single best approach. Combining elements from multiple proposals is allowed
   when it produces a stronger plan, but never as a compromise to avoid deciding. Hedging here costs
   real implementation time downstream.
4. **Concrete plan** — name every file to create or modify, the test strategy, and items you cannot
   resolve at design time (defer them explicitly).

## Adversarial Stress Test

Before presenting to the human, spawn the **adversarial** sub-agent using the Agent tool with
`subagent_type: "craft:adversarial"`. Pass it: the chosen plan + R1..Rn + key facts from
exploration.

Include the adversarial finding in the output verbatim. If severity is P0: add a `[P0-BLOCKER]`
marker above the human gate and require the human to address it before approving, either by
adjusting the plan or explicitly accepting the risk.

## Output for Human Gate

```markdown
## Architecture Proposal

### Approaches Considered

**[Approach 1]:** [2–3 sentence summary of what this approach does and how]

**[Approach 2]:** [2–3 sentence summary] _(include only when weighed)_

**[Approach 3]:** [2–3 sentence summary] _(include only when weighed)_

### Tradeoffs

For each dimension where approaches meaningfully differ, describe the tradeoff as prose:

- **[Dimension]:** [Approach A] does [X], while [Approach B] does [Y]. [Why it matters.]
- **R3 coverage:** [Approach A] only partially covers this because [reason]. [Approach B] handles it
  fully via [mechanism].
- ...

_(One entry per requirement where approaches diverge in coverage. Omit requirements all approaches
cover identically.)_

### Recommendation: [approach name]

**Why this approach:** [1–2 paragraphs explaining why this approach wins given the requirements and
codebase context. Reference specific tradeoffs above.]

**Concrete implementation differences vs. alternatives:**

- [Alternative]: [specific difference — e.g., "adds an adapter layer in `pkg/auth/` that Minimal
  avoids by reusing the existing middleware directly"]
- ...

### Implementation Plan

Files to create/modify:

- `path/to/file` — [what and why]

Test strategy: [what to test and how]

### Deferred to Implementation

- [ ] [item] — [why it couldn't be resolved here]

### Adversarial Finding

**Severity:** [P0 / P1 / P2 / none] [Finding text] **Applies to:** [whether this risk is specific to
the recommended approach or shared across all] **Mitigation:** [how addressed in plan, or "accepted
risk with rationale"]
```

## Human Gate

Present the output above, then output the following verbatim:

```markdown
## ⏸ HUMAN GATE — Architecture

Review the proposal above. **yes** to proceed, or **adjust** followed by the desired changes.
```

If the adversarial finding is P0, append this line:

```markdown
**⚠ The adversarial finding is P0-BLOCKER** — address it before approving, either by adjusting the
plan or explicitly accepting the risk.
```

Wait for the human to respond before proceeding.

After approval: write the full architectural decision to `workpad.md` under
`## Architectural Decision`. The implement phase requires every subsection to proceed: approaches
considered, tradeoffs, recommendation with rationale, concrete implementation differences, deferred
items, adversarial finding, **files to create/modify**, and **test strategy**.
