# dev-mesh-plugin

A Claude Code plugin marketplace for Dev Mesh — AI-powered integration across Azure DevOps, Jira, and Confluence.

## What ships

| Plugin | Description |
|---|---|
| `dev-mesh` | Ticket analysis, solution design generation, and intelligent PR review |

## Quick start

```
/dm-analyze ADO:12345
/dm-analyze NEXUS-456

/dm-design ADO:12345

/dm-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789

/dm-standup

/dm-sync ADO:12345

/dm-confluence-search "retry logic"
```

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| `AZURE_DEVOPS_EXT_PAT` env var | Code (Read) + Work Items (Read) + Pull Request Threads (Read) scopes |
| Atlassian credentials | First run prompts automatically via `mcp__claude_ai_Atlassian__authenticate` |

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
