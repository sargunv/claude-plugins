# Exploration Pass Contract

Use this contract for exploration sub-agents in `/craft-explore`.

Required output:

- `Key Facts` (<=20 bullets, dense and specific, with exact file paths and line numbers when
  relevant)
- `Code Snippets` (exact snippets only, never pseudocode, labeled with why they matter)
- `Key Files` (5-10 most important files with one-sentence explanation)
- `Appendix` (lower-priority observations)

Rules:

- Read actual files before making claims.
- Note important absences.
- Say when something is ambiguous.
- Do not make implementation recommendations.
- Stay within the assigned focus area.
- Fetch external docs only when local files are insufficient and the dependency or API is relevant.
