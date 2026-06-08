# dev-mesh-plugin

A Claude Code plugin for Dev Mesh — AI-powered integration with Azure DevOps.

## Installation

### Claude Code

Run these two commands inside Claude Code:

```
/plugin marketplace add godfreylab/dev-mesh-plugin
/plugin install dev-mesh
```

Then restart Claude Code and run `/mcp` to verify the MCP servers are connected.

## Quick start

```
/dm-ado-item-status 12345
/dm-ado-item-status https://dev.azure.com/<your-org>/<your-project>/_workitems/edit/12345

/dm-ado-pr-review https://dev.azure.com/<your-org>/<your-project>/_git/<repo>/pullrequest/789
```

### Skills

| Skill | What it does |
|---|---|
| `/dm-ado-item-status` | Full status snapshot of an ADO User Story: story details, parent, related items, PRs, branches, and commits |
| `/dm-ado-pr-review` | AI code review on an ADO PR — posts inline comments directly on the PR |

## Prerequisites

| Tool | Install |
|---|---|
| Node.js (LTS) | https://nodejs.org |
| Python + `uv` | https://docs.astral.sh/uv/getting-started/installation/ |

## MCP setup

MCP servers are defined in `.mcp.json`.

| Server | Package | Purpose |
|---|---|---|
| `ado` | `@azure-devops/mcp` (npx) | Azure DevOps — work items, PRs, pipelines |
| `mcp-atlassian` | `mcp-atlassian` (uvx) | Jira + Confluence |
| `CLI-Microsoft365` | `@pnp/cli-microsoft365-mcp-server` (npx) | Microsoft 365 CLI |

### Configuration

Copy `.claude/settings.local.json.example` to `.claude/settings.local.json` and fill in your credentials:

```jsonc
{
  "env": {
    "ADO_ORG": "your-ado-org",
    "JIRA_URL": "https://your-org.atlassian.net",
    "JIRA_USERNAME": "you@example.com",
    "JIRA_API_TOKEN": "<your-atlassian-api-token>",
    "CONFLUENCE_URL": "https://your-org.atlassian.net/wiki",
    "CONFLUENCE_USERNAME": "you@example.com",
    "CONFLUENCE_API_TOKEN": "<your-atlassian-api-token>"
  }
}
```

`settings.local.json` is gitignored — never commit real tokens.

### Verify the server loaded

After restarting Claude Code run `/mcp` — `ado` should show as connected.

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
