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
| Azure CLI | https://learn.microsoft.com/en-us/cli/azure/install-azure-cli |

## MCP setup

This plugin requires the ADO MCP server defined in `.mcp.json`.

| Server | Package | Purpose |
|---|---|---|
| `ado` | `@azure-devops/mcp` (npx) | Azure DevOps — work items, PRs, pipelines |

### Configuration

Copy `.claude/settings.local.json.example` to `.claude/settings.local.json` and fill in your values:

```jsonc
{
  "env": {
    "ADO_ORG": "your-ado-org"
  },
  "enabledMcpjsonServers": ["ado"]
}
```

`settings.local.json` is gitignored — never commit real tokens.

### Verify the server loaded

After restarting Claude Code run `/mcp` — `ado` should show as connected.

## Structure

See `plugins/dev-mesh/README.md` for the full component inventory.
See `CLAUDE.md` for contribution conventions.
