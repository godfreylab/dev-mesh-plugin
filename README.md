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
| Python + `uv` | https://docs.astral.sh/uv/getting-started/installation/ |

## MCP setup

This plugin requires two MCP servers defined in `.mcp.json`:

| Server | Package | Purpose |
|---|---|---|
| `ado` | `@azure-devops/mcp` (npx) | Azure DevOps — work items, PRs, pipelines |
| `mcp-atlassian` | `mcp-atlassian` (uvx) | Jira + Confluence |

### Configuration

Copy `.claude/settings.local.json.example` to `.claude/settings.local.json` and fill in your credentials:

```jsonc
// .claude/settings.local.json
{
  "env": {
    "ADO_ORG": "your-ado-org",                         // ADO organisation slug
    "JIRA_URL": "https://your-org.atlassian.net",
    "JIRA_USERNAME": "you@example.com",
    "JIRA_API_TOKEN": "<Atlassian API token>",          // https://id.atlassian.com/manage-profile/security/api-tokens
    "CONFLUENCE_URL": "https://your-org.atlassian.net/wiki",
    "CONFLUENCE_USERNAME": "you@example.com",
    "CONFLUENCE_API_TOKEN": "<Atlassian API token>"
  },
  "enabledMcpjsonServers": ["ado", "mcp-atlassian"]
}
```

`settings.local.json` is gitignored — never commit real tokens.

### Verify both servers loaded

After restarting Claude Code run `/mcp` — both `ado` and `mcp-atlassian` should show as connected.

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
