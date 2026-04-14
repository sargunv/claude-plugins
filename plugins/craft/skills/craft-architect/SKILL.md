---
name: craft-architect
description: Designs an implementation plan by running parallel architecture passes across distinct approach lenses, then stress-tests the recommendation adversarially. Requires requirements (R1..Rn) from a workpad or inline context. HUMAN GATE. Typically invoked by the /craft pipeline after clarification.
argument-hint: "[workpad path or inline context]"
---

# /craft-architect — Architecture

Arguments: $ARGUMENTS

---

## Role

Run thorough architectural analysis before a single line of code is written.

Do not read planning-pass or adversarial contract files in the orchestrator. Pass their paths to the
relevant sub-agents and have those sub-agents read them directly.

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context (task description, exploration report,
requirements R1..Rn). The human gate fires again — that is expected.

## Setup

Read the workpad (`workpad.md` or `$ARGUMENTS`) to retrieve:

- Task description
- Exploration Report (Key Facts + Key Files + code snippets)
- Requirements R1..Rn from clarify

If the workpad is missing or does not contain requirements R1..Rn, stop and ask for it. Do not
proceed without requirements — architecture without requirements produces plans that fail review.

Read the key files surfaced during exploration before spawning planning passes.

## Approach Passes

Spawn planning passes based on task scope:

**Small tasks** (≤3 requirements, single-file or tightly scoped changes): spawn **Minimal only**.
**Medium tasks** (cross-file, new component/function, 4–6 requirements): spawn **Minimal +
Clean/Robust**. **Large tasks** (new subsystem, API surface change, data model change, ≥7
requirements): spawn **Minimal + Clean/Robust + Pragmatic**.

Give each pass a distinct approach directive:

**Approach: Minimal** — fewest moving parts, smallest diff, maximum reuse of existing code.
**Approach: Clean/Robust** — correct abstractions, full error handling, future-proof design.
**Approach: Pragmatic** — best fit for this codebase's actual patterns, fastest to implement
correctly.

**Optional (any task size, spawn when warranted):** **Approach: Performance** — optimize for
throughput/latency; spawn when performance is a stated requirement. **Approach: Incremental** —
staged rollout with feature flags; spawn when change is high-risk or zero-downtime deployment is
required.

Each pass receives the same context: task + key facts + code snippets + R1..Rn. Each pass must
commit to exactly ONE approach. No hedging. No "it depends."

Require each pass to read and follow `architecture-approach-contract.md` next to this skill file.

## Synthesis

After all approach passes return:

1. **Per-pass summaries** — distill each approach into a 2-3 sentence summary capturing what it does
   and how.

2. **Tradeoffs comparison** — build a comparison table across approaches. Include dimensions where
   approaches meaningfully differ: scope of change, complexity, risk, and any requirements where
   coverage diverges. Omit dimensions where all approaches agree.

3. **Deferred to Implementation** — list the unknowns the approach passes could not resolve. These
   go in the workpad.

4. **Recommendation** — pick the single best approach. Combining elements from multiple approaches
   is allowed when it produces a stronger plan, but never as a compromise to avoid deciding.
   Include: rationale, concrete implementation differences vs. alternatives, files to create/modify,
   and test strategy.

## Adversarial Stress Test

After synthesis, run one adversarial stress-test pass against the recommendation.

Pass it: the synthesized recommendation + R1..Rn + key facts.

Read and follow `architecture-adversarial-contract.md` next to this skill file.

Include the adversarial finding in the output. If P0: add `[ADVERSARIAL-P0]` marker. The human
should address a P0 finding before approving. P1/P2 findings are informational.

## Output for Human Gate

```markdown
## Architecture Proposal

### Approaches Considered

**Minimal:** [2-3 sentence summary of what this approach does and how]

**Pragmatic:** [2-3 sentence summary] _(omit if not spawned)_

**Clean/Robust:** [2-3 sentence summary] _(omit if not spawned)_

**Performance:** [2-3 sentence summary] _(include if spawned)_

**Incremental:** [2-3 sentence summary] _(include if spawned)_

### Tradeoffs

For each dimension where approaches meaningfully differ (scope, complexity, risk, requirement
coverage), describe the tradeoff as prose:

- **[Dimension]:** [Approach A] does [X], while [Approach B] does [Y]. [Why it matters.]
- **R3 coverage:** [Approach A] only partially covers this because [reason]. [Approach B] handles it
  fully via [mechanism].
- ...

_(Include an entry for each requirement where approaches diverge in coverage. Omit requirements all
approaches cover identically.)_

### Recommendation: [approach name]

**Why this approach:** [1-2 paragraphs explaining why this approach wins given the requirements and
codebase context. Reference specific tradeoffs from the table above.]

**Concrete implementation differences vs. alternatives:**

- [Alternative approach]: [specific difference — e.g., "adds an adapter layer in `pkg/auth/` that
  Minimal avoids by reusing the existing middleware directly"]
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

Review the proposal above. **approve** to proceed, or **adjust** followed by the desired changes.
```

If the adversarial finding is P0, append this line:

```markdown
**⚠ The adversarial finding is P0-BLOCKER** — address it before approving, either by adjusting the
plan or explicitly accepting the risk.
```

Wait for the human to respond before proceeding.

After approval: write the full architectural decision to `workpad.md` under
`## Architectural Decision`. The implement phase requires every subsection to proceed: approaches
considered, tradeoffs comparison, recommendation with rationale, concrete implementation
differences, deferred items, adversarial finding, **files to create/modify**, and **test strategy**.
