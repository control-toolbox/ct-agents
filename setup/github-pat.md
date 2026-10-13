# GitHub MCP server & personal access tokens

The GitHub MCP server (`mcp__github__*` tools: `add_issue_comment`, `create_pull_request`,
etc.) authenticates with a **fine-grained personal access token (PAT)** read from the
macOS Keychain by a wrapper script (`~/bin/github-mcp.sh`):

```zsh
#!/bin/zsh
export GITHUB_PERSONAL_ACCESS_TOKEN=$(security find-generic-password -s "github-mcp" -a "claude" -w)
exec npx -y @modelcontextprotocol/server-github
```

Fine-grained PATs have three gotchas that all surface as the same opaque error —
`MCP error -32603: Permission Denied: Resource not accessible by personal access token`
— with no indication of which one is at fault:

1. **"Issues" permission, not just "Pull requests".** Commenting on a PR
   (`add_issue_comment`) goes through GitHub's **Issues** API under the hood, even
   though it appears on a pull request. A token configured with only "Pull requests:
   Read and write" will fail; it also needs **"Issues: Read and write"**.
2. **"Repository access: all repositories owned by you" excludes organization repos.**
   A fine-grained PAT's "Resource owner" is fixed at creation time to either your
   personal account or an organization you belong to. If it's your personal account,
   "all repositories" never includes org-owned repos (e.g. anything under
   `control-toolbox/*`), no matter the permissions granted. To reach an org repo, create
   a **separate token with Resource owner = the organization**.
3. **Organization approval.** If the org has "Require administrator approval" enabled
   (org Settings → Third-party Access → Personal access tokens), every org-scoped
   fine-grained PAT sits as a **pending request** until an owner approves it at
   `https://github.com/organizations/<org>/settings/personal-access-tokens` (Pending
   requests tab) — the token is otherwise unusable even with correct permissions.

**Diagnosing without writing anything**: `GET /repos/{owner}/{repo}` (e.g. via `gh api`)
returns a `permissions` object, but for a **public** repo this reflects the
*authenticated account's* role, not what a *specific fine-grained token* is scoped to —
reads on public repos succeed almost regardless of token scope. There is no reliable
read-only way to verify a fine-grained PAT's write access short of attempting the write
(or inspecting the token's settings page directly); a throwaway comment that gets
deleted afterwards is an acceptable test.

**Fallback that keeps working**: the `gh` CLI typically authenticates via a classic
OAuth token (`gh auth status` shows its scopes, e.g. `repo`, `admin:org`), which is not
subject to the fine-grained per-org restrictions above. When the GitHub MCP server
fails, retry the same action with `gh pr comment` / `gh issue comment` / `gh api` before
concluding it's a broken feature rather than a token issue.

**Rotating the token**: after fixing permissions/resource-owner/approval on
github.com, update the Keychain entry (`-U` overwrites in place) and reconnect the MCP
server (`/mcp` → reconnect, or restart the session) so the wrapper script picks up the
new value:

```bash
security add-generic-password -U -s github-mcp -a claude -w '<new_token>'
```

