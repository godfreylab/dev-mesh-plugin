# dev-mesh

An AI-powered Claude Code plugin that connects **Azure DevOps**, **Jira**, and **Confluence** to accelerate engineering workflows.

## Components

| Component | Path | Purpose |
|---|---|---|
| `dm-analyze` skill | `skills/dm-analyze/` | Pull and analyze a ticket from ADO or Jira |
| `dm-design` skill | `skills/dm-design/` | Generate a solution design doc from ticket context |
| `dm-review` skill | `skills/dm-review/` | Review a PR with linked ticket + Confluence context |
| `dm-standup` skill | `skills/dm-standup/` | Summarize recent work items and PRs for standup |
| `dm-sync` skill | `skills/dm-sync/` | Cross-reference ADO ↔ Jira ↔ Confluence |
| `dm-confluence-search` skill | `skills/dm-confluence-search/` | Search Confluence for docs relevant to current work |
| `ado-reader` agent | `agents/ado-reader.md` | ADO data fetcher |
| `jira-reader` agent | `agents/jira-reader.md` | Jira data fetcher |
| `confluence-reader` agent | `agents/confluence-reader.md` | Confluence data fetcher |
| `solution-designer` agent | `agents/solution-designer.md` | Solution design generator |
| `pr-reviewer` agent | `agents/pr-reviewer.md` | PR review engine |
| `impact-analyzer` agent | `agents/impact-analyzer.md` | Change blast-radius analyzer |

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| `npx` | Included with Node.js |
| ADO PAT | Set `AZURE_DEVOPS_EXT_PAT` with **Code (Read)**, **Work Items (Read)**, **Pull Request Threads (Read)** scopes |
| Atlassian credentials | Configure via `mcp__claude_ai_Atlassian__authenticate` on first run |

## Usage

```
/dm-analyze ADO:12345
/dm-analyze NEXUS-456

/dm-design ADO:12345

/dm-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-standup

/dm-sync ADO:12345

/dm-confluence-search "payment gateway retry logic"
```
