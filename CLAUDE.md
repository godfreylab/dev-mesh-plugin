# Contributing to dev-mesh-plugin

Per-repo guidance for anyone (human or agent) editing this repo. Loaded by Claude Code when you work inside `dev-mesh-plugin/`.

## What this repo is

A Claude Code plugin that connects Azure DevOps, Jira, and Confluence. It ships under the `dev-mesh-marketplace` marketplace and installs as the `dev-mesh` plugin.

See `plugins/dev-mesh/CLAUDE.md` for plugin-level guidance that applies when using the plugin.

## Conventions

1. **Skill SKILL.md files are procedural runbooks.** Write numbered steps with exact MCP tool call signatures. Keep the `description` frontmatter behavioral and specific — Claude uses it to decide when to auto-invoke.
2. **Agent files are system prompts.** Write in second person to the agent. Include hard constraints, output format, and what the agent does NOT do.
3. **No README.md inside `agents/`.** The loader treats every `.md` file in `agents/` as an agent definition. Contribution guidance lives in this CLAUDE.md and in `plugins/dev-mesh/AGENTS.md`.
4. **Security by default.** Every agent and skill must include inline comments on auth/input-handling logic. Follow OWASP Top 10 practices in any generated code. Never log or output credentials, tokens, or PII.
5. **Read the plugin CLAUDE.md.** It defines MCP namespaces, defaults (ADO org, project), and data handling rules that all skills and agents must follow.
6. **Skill naming: `dm-*`.** All skills in this plugin use the `dm-` prefix.

## Validating changes

```powershell
# Validate all JSON files are well-formed
Get-ChildItem -Recurse -Filter *.json | ForEach-Object { $_ | Get-Content | ConvertFrom-Json | Out-Null; Write-Host "OK: $($_.FullName)" }

# Validate the plugin manifest
claude plugin validate .
```
