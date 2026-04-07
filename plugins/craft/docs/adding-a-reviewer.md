# Adding a Reviewer

Adding a reviewer requires two files: the agent definition and a row in the topic-reviewer map.

---

## Create the agent file

Create `agents/reviewer-<domain>.md`, where `<domain>` is a short lowercase label for the review
area (e.g., `accessibility`, `i18n`, `concurrency`, `migrations`). The domain name becomes the
reviewer's identifier in the topic-reviewer map and in review reports.

Start with frontmatter:

```yaml
---
name: reviewer-<domain>
description: Reviews code for [specific failure modes this reviewer flags].
model: inherit
color: blue
tools: Read, Glob, Grep, WebFetch, WebSearch
---
```

The body must contain four sections. See [reviewer-contract.md](../references/reviewer-contract.md)
for the full specification of each section and the standard finding format.

**`## My Domain (Authoritative)`** — List every class of issue this reviewer flags. Be specific:
name the exact failure modes, not broad categories. Other reviewers use this list to determine
whether a finding is yours or theirs.

**`## What I Do NOT Flag`** — List every domain this reviewer defers to, using the format
`issue type → reviewer-name`. This section is a hard instruction to the agent, not a suggestion.
Without it, reviewers drift into each other's territory and the same finding appears multiple times.

**`## Confidence Definitions`** — State what CERTAIN and LIKELY mean for this specific domain. The
generic definitions in [reviewer-contract.md](../references/reviewer-contract.md) give the baseline;
your section narrows them to what is actually achievable given the evidence this reviewer can
access.

**`## Finding Format`** — State that the reviewer uses the standard format from
[reviewer-contract.md](../references/reviewer-contract.md). Add domain-specific fields here if your
reviewer needs them. For example, `reviewer-security` adds a `CWE/OWASP` field.

---

## Register in the topic-reviewer map

Add a row to the table in [topic-reviewer-map.md](../skills/craft-review/topic-reviewer-map.md):

- **Tag** — the tag name the topic-tagger will emit (e.g., `accessibility`).
- **Description** — one sentence describing what code changes should trigger this tag. The
  topic-tagger reads this column to decide whether to emit the tag. Be specific: "ARIA attributes
  missing or incorrect, color contrast violations, keyboard navigation broken" is more useful than
  "accessibility issues."
- **Always?** — leave blank for conditional reviewers. Mark `✓` only if this reviewer should run on
  every diff regardless of content. Use this sparingly; always-on reviewers add cost to every
  review.
- **Reviewer(s)** — `reviewer-<domain>`.

If your reviewer covers a topic that an existing tag already captures, add it to the Reviewer(s)
column of that row instead of creating a new tag.

No other files need changing. The topic-tagger picks up the new tag automatically from the
Description column on the next run. The review orchestrator activates the reviewer whenever the
tagger emits the tag.

---

## Test it

Run `/craft-review` on a branch whose diff clearly falls within the new reviewer's domain. Check the
review report for the reviewer's name in the "Reviewers activated" list. If the reviewer does not
appear, the topic-tagger did not emit the tag — re-read the Description column in the map and
confirm the diff contains the kind of change the description targets.

---

## Tips

**Define the domain by listing failure modes, not topics.** "Flags: missing ARIA labels, incorrect
role attributes, keyboard traps, insufficient color contrast" gives the agent a testable checklist.
"Reviews accessibility" does not.

**Write the exclusion list before writing the domain.** Knowing what the reviewer does NOT flag
clarifies the domain boundary and prevents overlap with existing reviewers. Read the other reviewer
files first.

**Most findings are P2 or P3.** Reserve P1 for issues that block a reasonable user from completing a
task. P0 is for issues that cause data loss, security breaches, or crashes in production. Reviewers
that over-use P0 and P1 lose credibility quickly.

**Don't gate confidence definitions too tightly.** The pipeline independently verifies all findings
below CERTAIN confidence, so reviewers don't need to self-censor uncertain findings.
