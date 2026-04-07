---
name: craft-architect
description: Designs an implementation plan by running parallel architecture agents (debate + single-approach), then stress-tests the synthesis adversarially. Requires requirements (R1..Rn) from a workpad or inline context. HUMAN GATE. Typically invoked by the /craft pipeline after clarification.
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

If the workpad is missing or does not contain requirements R1..Rn, stop and ask for it. Do not
proceed without requirements — architecture without requirements produces plans that fail review.

Read the key files surfaced during exploration before spawning agents.

## Track 1 — Party Mode Debate

Spawn **one** Party Mode agent using the Agent tool with `subagent_type: "craft:architect-party"`.

Pass to the agent:

- Task description
- Exploration key facts and relevant code snippets
- Requirements R1..Rn

The agent runs three sequential personas (PM → Architect → Developer) in one session and produces a
**Tradeoff Map**:

- Convergences (all personas agreed)
- Divergences (unresolved tensions between personas)
- Deferred to Implementation (unknowns the developer persona flagged)

## Track 2 — Classical Single-Approach Agents

Spawn based on task scope:

**Small tasks** (≤3 requirements, single-file or tightly scoped changes): spawn **Minimal only**.
**Medium tasks** (cross-file, new component/function, 4–6 requirements): spawn **Minimal +
Pragmatic**. **Large tasks** (new subsystem, API surface change, data model change, ≥7
requirements): spawn **Minimal + Clean/Robust + Pragmatic**.

All agents use `subagent_type: "craft:architect"`. Pass each a distinct approach directive in the
prompt:

**Agent: Minimal** — fewest moving parts, smallest diff, maximum reuse of existing code. **Agent:
Clean/Robust** — correct abstractions, full error handling, future-proof design. **Agent:
Pragmatic** — best fit for this codebase's actual patterns, fastest to implement correctly.

**Optional (any task size, spawn when warranted):** **Agent: Performance** — optimize for
throughput/latency; spawn when performance is a stated requirement. **Agent: Incremental** — staged
rollout with feature flags; spawn when change is high-risk or zero-downtime deployment is required.

Each agent receives the same context as Track 1 (task + key facts + code snippets + R1..Rn). Each
commits to exactly ONE approach. No hedging. No "it depends."

## Synthesis

After all Track 1 and Track 2 agents return:

1. **Convergences** — where Party Mode personas AND ≥2 classical agents agree. Mark them confident.

2. **Divergences** — genuine choices where agents disagree. For each: present the tradeoff, state
   a recommendation with rationale.

3. **Deferred to Implementation** — from the Party Mode developer persona plus any unknowns
   classical agents could not resolve. List explicitly. These go in the workpad.

4. **Recommended approach** — one concrete synthesis recommendation. Not a committee compromise. An
   actual decision with rationale. Include: files to create/modify, test strategy, requirement
   coverage list (R1..Rn × approach).

## Adversarial Stress Test

After synthesis, spawn the adversarial agent using the Agent tool with
`subagent_type: "craft:architect-adversarial"`.

Pass it: the synthesized recommendation + R1..Rn + key facts.

The adversarial agent finds the single biggest structural flaw — the one assumption that, if wrong,
causes the plan to fail. It returns a finding with severity P0/P1/P2/none.

Include the adversarial finding in the output. If P0: add `[ADVERSARIAL-P0]` marker. The human
should address a P0 finding before approving. P1/P2 findings are informational.

## Output for Human Gate

```markdown
## Architecture Proposal

### Recommended Approach: [name]

[2-paragraph description]

### Requirement Coverage

- R1: [statement] — [covered by]: [how]
- R2: ...

### Key Trade-offs Resolved

- [Divergence]: [Decision] — [Rationale]

### Implementation Plan

Files to create/modify:

- `path/to/file` — [what and why]

Test strategy: [what to test and how]

### Deferred to Implementation

- [ ] [item] — [why it couldn't be resolved here]

### Adversarial Finding

**Severity:** [P0 / P1 / P2 / none] [Finding text] **Mitigation:** [how addressed in plan, or
"accepted risk with rationale"]
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
`## Architectural Decision`. Include ALL subsections: chosen approach, requirement coverage,
trade-offs, deferred items, adversarial finding, **files to create/modify**, and **test strategy**.
The implement phase requires all of these to proceed.
