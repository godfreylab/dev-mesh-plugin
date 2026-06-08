---
name: dm-ado-pr-review
description: >
  Perform a thorough code review on an Azure DevOps pull request and post inline comments directly
  on the PR for every issue found. Use when the user shares an ADO PR link or PR ID and asks for
  a code review, says "review this PR", "code review PR #N", "check this pull request",
  "dm-ado-pr-review <link>", "leave review comments on PR", or shares any Azure DevOps PR URL.
  Fetches all changed files, analyzes them for bugs, security vulnerabilities (OWASP Top 10),
  code duplication, and maintainability issues, then posts inline ADO PR thread comments via MCP
  on the exact lines that need attention. Always prefer this skill over ad-hoc code review
  whenever a PR link or PR ID is mentioned. Requires a live ADO MCP connection.
---

# dm-ado-pr-review

Fetch an Azure DevOps pull request, review every changed file for code quality issues, and post
inline comments directly on the PR.

## Defaults

- **ADO organisation**: read from the `ADO_ORG` environment variable — ask the user if not set
- **ADO project**: ask the user if not specified and cannot be parsed from the input URL
- **Output directory**: `reviews/`

---

## Step 0 — Verify connection

Read the ADO organisation from the `ADO_ORG` environment variable. If it is empty or unset,
stop and tell the user:

> `ADO_ORG` is not configured. Set it in `.claude/settings.local.json` under `env.ADO_ORG`.

Verify the ADO MCP server is reachable by calling `mcp__ado__core_list_projects` with
`{ "organization": "<ADO_ORG>" }`. If it fails, stop and tell the user:

> Cannot reach the Azure DevOps MCP server. Ensure your `AZURE_DEVOPS_EXT_PAT` or Azure CLI
> session is active, then restart your terminal.

Also verify `az account show` succeeds — the REST API calls in Step 3–4 use `az` for auth.
If `az` is not available, set `$usePatAuth = $true` and read `AZURE_DEVOPS_EXT_PAT` from
the environment instead (used as a Basic auth header below).

---

## Step 1 — Parse the input

Accept any of:
- ADO PR URL: `https://dev.azure.com/<org>/<project>/_git/<repo>/pullrequest/<id>`
- Bare integer: `123` (ask the user which repo if not obvious from context)
- Repo+ID shorthand: `repo-name#123`

Extract: **org**, **project**, **repository name**, **PR ID (integer)**.

If the repository name cannot be determined, call `mcp__ado__repo_list_repos_by_project` with
`{ "project": "<project>" }` and ask the user to confirm.

---

## Step 2 — Fetch PR metadata

Call `mcp__ado__repo_get_pull_request_by_id`:
```json
{
  "repositoryId": "<repo-name>",
  "pullRequestId": <pr_id>,
  "includeWorkItemRefs": true
}
```

Record:
- `title`, `description`
- `repository.id` — the repo GUID, required for all subsequent API calls
- `sourceRefName` → strip `refs/heads/` prefix for use as the branch name
- `targetRefName` → strip `refs/heads/` prefix
- `createdBy.displayName`
- `status` — abort if `Abandoned` or `Completed` (ask user to confirm if they still want a review)
- `workItemRefs` — linked work items (optional context, not required)

---

## Step 3 — Get changed files via ADO REST API

Use PowerShell to fetch the list of files changed in the latest PR iteration:

```powershell
# --- Auth: prefer Azure CLI; fall back to PAT env var ---
# Azure CLI token scope for ADO: 499b84ac-1321-427f-aa17-267ca6975798
if (-not $usePatAuth) {
    $token = az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798 `
             --query accessToken -o tsv
    $headers = @{ Authorization = "Bearer $token" }
} else {
    # Basic auth with PAT — base64-encode ":$pat"
    $pat = $env:AZURE_DEVOPS_EXT_PAT
    $encoded = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))
    $headers = @{ Authorization = "Basic $encoded" }
}

$base = "https://dev.azure.com/$org/$project/_apis/git/repositories/$repoId"

# Find the latest PR iteration (each push creates a new iteration)
$iters = Invoke-RestMethod "$base/pullRequests/$prId/iterations?api-version=7.1" -Headers $headers
$iterationId = ($iters.value | Sort-Object id -Descending | Select-Object -First 1).id

# Retrieve file change list for this iteration
$changes = Invoke-RestMethod "$base/pullRequests/$prId/iterations/$iterationId/changes?api-version=7.1" -Headers $headers
$changedFiles = $changes.changeEntries |
    Where-Object { $_.changeType -ne "delete" } |
    Select-Object -ExpandProperty item |
    Select-Object -ExpandProperty path
```

**Filter out files that don't benefit from human review:**
- Auto-generated: `*.g.cs`, `*.Designer.cs`, `**/Migrations/**`, `**/generated/**`, `**/dist/**`
- Lock files: `package-lock.json`, `yarn.lock`, `*.lock`, `*.sum`
- Binary / asset files: images, fonts, compiled binaries

If more than 20 reviewable files remain, prioritise: source code files first, then config files.
Inform the user: "This PR has N files — reviewing the top 20 most impactful."

---

## Step 4 — Fetch file content from source branch

For each file to review, retrieve its content from the **source (PR) branch**:

```powershell
$sourceBranch = "<sourceRefName stripped of refs/heads/>"
$encodedPath  = [Uri]::EscapeDataString($filePath)  # safe-encode the file path

$content = Invoke-RestMethod `
  "$base/items?path=$encodedPath&versionDescriptor.versionType=branch&versionDescriptor.version=$sourceBranch&`$format=text&api-version=7.1" `
  -Headers $headers -ContentType "text/plain"
```

If a file exceeds 500 lines, fetch only the sections that appear in the PR diff. To get line-level
diff context for a specific file, you can call:

```powershell
$fileDiff = Invoke-RestMethod `
  "$base/pullRequests/$prId/iterations/$iterationId/changes?api-version=7.1" `
  -Headers $headers
# Then filter $fileDiff.changeEntries by the specific $filePath
```

---

## Step 5 — Review each file

Work through each file. Tailor depth to the language (C#, TypeScript, Python, Go, Java, etc.) —
apply the universal patterns below plus any language-specific idioms you know.

### What to look for (in priority order)

**🔴 Must fix** — post thread with `status: "Active"`
- **Security (OWASP Top 10):** injection (SQL, command, LDAP), broken auth, missing access
  control checks, XSS, insecure deserialisation, sensitive data in logs/responses, SSRF,
  hardcoded credentials or API keys
- **Logic errors:** null/undefined dereferences, off-by-one errors, race conditions, incorrect
  boolean logic, unreachable code that masks a bug
- **Data integrity:** missing transaction boundaries, silent data loss, incorrect error handling
  that swallows failures

**🟡 Should fix** — post thread with `status: "Active"`
- **Code duplication:** identical or near-identical blocks that belong in a shared
  function/method — note the other location(s) so the author can consolidate
- **Missing error handling:** unhandled exceptions at system boundaries, swallowed errors,
  missing null checks at API entry points
- **Performance:** N+1 query patterns, unnecessary full-collection loads, synchronous I/O where
  async is expected, unbounded loops over external data

**🟢 Consider** — post thread with `status: "Pending"`
- Overly complex conditionals or deeply nested logic (cyclomatic complexity > 10)
- Naming that obscures intent (single-letter variables outside loops, misleading method names)
- Dead code or unused imports

**💡 Non-blocking suggestion** — post thread with `status: "Pending"`
- Small refactors that improve clarity without changing behaviour
- Opportunities to use existing utilities already in the codebase

**Do NOT comment on:** formatting, whitespace, indentation, or anything a linter handles.

### Recording findings

As you review, build a list of findings with:
- `filePath` — full path from repo root (e.g. `/src/auth/TokenService.cs`)
- `lineNumber` — line in the **new (right) file** where the issue is located (1-based)
- `severity` — one of: `must-fix`, `should-fix`, `consider`, `suggestion`
- `category` — e.g. `Security`, `Duplication`, `Logic`, `Performance`, `Naming`
- `message` — 2–4 sentences: what the problem is, why it matters, how to fix it
- `codeExample` — optional: a before/after code snippet (fenced block, ≤15 lines)

---

## Step 6 — Check for existing threads (avoid duplicates)

Before posting, call `mcp__ado__repo_list_pull_request_threads`:
```json
{
  "repositoryId": "<repo-guid>",
  "pullRequestId": <pr_id>
}
```

Build a set of already-commented `(filePath, lineNumber)` pairs from the existing threads.
Skip any finding whose exact `(filePath, lineNumber)` already has an active thread, to avoid
flooding the PR if this skill is run multiple times.

---

## Step 7 — Post inline comments via ADO MCP

For each finding not already covered, call `mcp__ado__repo_create_pull_request_thread`:

```json
{
  "repositoryId": "<repo-guid>",
  "pullRequestId": <pr_id>,
  "project": "<project>",
  "filePath": "<filePath from finding>",
  "rightFileStartLine": <lineNumber>,
  "rightFileStartOffset": 1,
  "rightFileEndLine": <lineNumber>,
  "rightFileEndOffset": 100,
  "content": "<formatted comment — see below>",
  "status": "<Active for 🔴🟡 | Pending for 🟢💡>"
}
```

> **Important:** `rightFileStartOffset` and `rightFileEndOffset` are required when specifying
> line positions — the API rejects offset `0`. Use `1` for start and `100` for end as safe defaults.

**Comment format:**

```
🤖 *Generated by AI (dm-ado-pr-review)*

🔴 **Must fix — Security: SQL Injection**

The query on this line builds SQL by concatenating user input, which allows an attacker to
manipulate the query structure. Use parameterised queries to separate code from data.

```csharp
// Vulnerable — do not do this:
var sql = $"SELECT * FROM users WHERE email = '{email}'";

// Safe — use parameters:
cmd.CommandText = "SELECT * FROM users WHERE email = @email";
cmd.Parameters.AddWithValue("@email", email);
```
```

Every comment must start with `🤖 *Generated by AI (dm-ado-pr-review)*` on the first line so
reviewers immediately see it is AI-generated. Never omit this header.

---

## Step 8 — Save the review report

Sanitise the PR title for a filename: lowercase, replace non-alphanumeric chars with hyphens,
collapse consecutive hyphens, truncate to 60 chars.

Write a markdown report to `reviews/<pr_id>-<sanitised-title>.md`:

```markdown
# PR Review: [#<pr_id>] <title>

> Reviewed on <YYYY-MM-DD> | Author: <createdBy> | Branch: `<source>` → `<target>`

## Summary

<3–5 sentences: overall code quality, most significant concerns, and your recommendation.>

## Findings

| # | File | Line | Severity | Category | Summary |
|---|---|---|---|---|---|
| 1 | `/src/auth/TokenService.cs` | 42 | 🔴 Must fix | Security | SQL injection via string concat |

## Recommendation

**[Approve / Approve with comments / Request changes]**

<1–2 sentences justifying the recommendation.>

---
_Generated by dm-ado-pr-review · <YYYY-MM-DD>_
```

Create the `reviews/` directory if it does not exist.

---

## Step 9 — Present to chat

Output to chat in this order:
1. **Summary** paragraph
2. Finding counts: `🔴 N must-fix · 🟡 N should-fix · 🟢 N consider · 💡 N suggestions`
3. Any 🔴 must-fix findings listed by name (so the author sees them immediately)
4. **Recommendation** (Approve / Approve with comments / Request changes)
5. `Inline comments posted to PR #<id>` and path to the saved report
