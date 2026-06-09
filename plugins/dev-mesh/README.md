# dev-mesh

An AI-powered Claude Code plugin for Azure DevOps workflows.

## Components

### Skills

| Skill | Path | Purpose |
|---|---|---|
| `dm-ado-item-status` | `skills/dm-ado-item-status/` | Full status snapshot of an ADO User Story — details, parent, related items, PRs, branches, commits. Saves to `docs/`. |
| `dm-ado-pr-review` | `skills/dm-ado-pr-review/` | AI code review on an ADO PR — posts inline comments directly on the PR. Saves a report to `reviews/`. |
| `dm-analyze` | `skills/dm-analyze/` | End-to-end solution design for an ADO User Story — scans linked repos, Jira issues, and Confluence pages; produces story breakdown, API contracts, DB migration plan, and workflow diagrams. Saves to `docs/`. |

### Agents

| Agent | Path | Purpose |
|---|---|---|
| `ado-reader` | `agents/ado-reader.md` | ADO data fetcher |
| `jira-reader` | `agents/jira-reader.md` | Jira data fetcher |
| `confluence-reader` | `agents/confluence-reader.md` | Confluence data fetcher |
| `solution-designer` | `agents/solution-designer.md` | Solution design generator |
| `pr-reviewer` | `agents/pr-reviewer.md` | PR review engine |
| `impact-analyzer` | `agents/impact-analyzer.md` | Change blast-radius analyzer |

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| ADO PAT | Set `AZURE_DEVOPS_EXT_PAT` with **Code (Read)**, **Work Items (Read)**, **Pull Request Threads (Read & Write)** scopes |

## Usage

```
/dm-ado-item-status 12345
/dm-ado-item-status https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345

/dm-ado-pr-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-analyze 12345
/dm-analyze https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345
```
