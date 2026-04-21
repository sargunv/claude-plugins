---
name: craft-push
description: Use after craft-refine in the craft pipeline. Pushes the current branch and opens a draft PR/MR using whatever forge tooling is available (CLI or MCP), then handles wrap-up (filing deferred findings in the team's tracker, cleaning up the workpad). HUMAN GATE before the PR opens.
argument-hint: "[workpad path]"
allowed-tools: Agent Bash Read Write Edit Glob Grep
---

# /craft-push — Push and Open PR

Arguments: $ARGUMENTS

---

## Role

Turn the fix-clean branch into a draft PR/MR, then wrap up loose ends (deferred findings, workpad
cleanup).

## Resuming

If `$ARGUMENTS` is a workpad path, read it to restore context (branch, deferred items, task
description, architectural decision, review findings).

## Step 1 — Prepare PR Body

Draft the PR/MR title and body. If the repo has a pull-request or merge-request template, follow it.
Common locations:

- GitHub: `.github/pull_request_template.md`
- GitLab: `.gitlab/merge_request_templates/Default.md`
- Forgejo / Codeberg: `.forgejo/pull_request_template.md` (or `.gitea/pull_request_template.md`)

Fall back to the following three-section structure if no template exists.

- **What** — files changed, high-level what.
- **Why** — placeholder for your human collaborator to fill in.
- **Test Plan** — how the changes were verified (tests added, manual checks).

## Step 2 — Human Gate

Present the proposed PR/MR title and body, then output the following verbatim:

```markdown
## ⏸ HUMAN GATE — PR Body

Review the proposed PR above. **yes** to push and open the draft, or **adjust** followed by the
desired changes.
```

Wait for the human to respond before proceeding.

## Step 3 — Push and Open Draft PR

After human approval, push the branch and open a draft PR/MR using the project's forge tooling:

```bash
git push -u origin <branch>
```

Then open the draft using whatever forge tooling is available — a CLI (for example,
`gh pr create --draft` on GitHub) or an MCP server for the host.

If no forge tooling is available or it fails, output the title and body so the human can open the
PR/MR manually. Record the URL (or failure reason) in the workpad. Update Phase Log: pr → done.

## Step 4 — Wrap-up

If deferred findings exist in the workpad, ask whether to file them in the team's issue tracker:

```markdown
## Wrap-up

**Deferred findings:** This task deferred [N] items.

- [P1] (review) Missing transaction on multi-table write
- [P2] (implement) Inconsistent error response shape
- ...

Would you like to file these? If so, which tracker (GitHub Issues, GitLab Issues, Codeberg/Forgejo
Issues, Linear, Jira, etc.)?

**Workpad:** The workpad at `workpad.md` contains the full task history (requirements, architecture
decision, review findings, deferred items). Would you like to keep it, or clean it up?
```

If no deferred findings exist, skip the deferred section and only ask about the workpad.

Act on the human's response — file tickets via whatever's available for the tracker (a CLI like
`gh issue create`, or an MCP server), clean up or preserve the workpad as requested.
