---
name: onboarding
description: "Run post-install setup checks for the drupal-sdlc plugin. Verifies GITHUB_TOKEN, MCP connectivity, and project config files. Auto-creates .mcp.json, drupal-claude.yml, and CLAUDE.md if missing. Use after installing the plugin on a new project, or when MCPs show as failed."
argument-hint: "(no arguments needed)"
---

# Plugin Onboarding

Run all setup checks and fix what's missing. Go through every check in order. Do not skip any.

## Project Context (if drupal-claude.yml exists)
```yaml
!`cat drupal-claude.yml 2>/dev/null || echo "not found"`
```

---

## Instructions

You are verifying that the drupal-sdlc plugin is correctly set up for this project. Run each check, report the result clearly, and provide the exact fix for anything that fails.

Use this format for each check:
- Pass: **Check N — Name**: one-line result
- Fail: **Check N — Name**: what's wrong + exact fix

At the end, show a summary table and tell the user what to do next.

---

### Check 0 — Node.js version (BLOCKER)

Run:
```bash
node --version 2>/dev/null || echo "NOT FOUND"
```

Extract the major version number. It must be **18 or higher**.

**If 18+:** Pass. Continue.

**If below 18 or NOT FOUND:** STOP. This is a hard blocker — nothing else will work.

Tell the user:
> Node.js 18+ is required. The MCP servers (Playwright and GitHub) cannot start on older versions.
>
> If you use nvm:
> ```shell
> nvm install 20
> nvm alias default 20
> ```
> If you don't have nvm, install Node.js from https://nodejs.org
>
> Then **restart Claude Code** and re-run `onboarding`.

Do NOT continue to the remaining checks if Node is too old.

---

### Check 1 — GITHUB_TOKEN

Run:
```bash
if [ -n "$GITHUB_TOKEN" ]; then echo "SET"; else echo "MISSING"; fi
```

**If SET:** Pass. Continue.

**If MISSING:** Fail. Tell the user:

> `GITHUB_TOKEN` is not set. This is the standard GitHub personal access token used by both the `gh` CLI and the GitHub MCP server.
>
> 1. Go to https://github.com/settings/tokens
> 2. Generate a classic token with `repo` + `read:org` scopes
> 3. Add it to your shell:
> ```shell
> echo 'export GITHUB_TOKEN=ghp_your_token_here' >> ~/.zshrc
> source ~/.zshrc
> ```
> Then **restart Claude Code completely** and re-run `onboarding`.

Do NOT stop here — continue with the remaining checks so the user sees the full picture.

---

### Check 2 — .mcp.json at project root

Run:
```bash
cat .mcp.json 2>/dev/null | head -1 || echo "NOT FOUND"
```

**If found:** Pass. Continue.

**If NOT FOUND:** Auto-create it from the plugin template.

Find the template:
```bash
ls ~/.claude/plugins/cache/axelerant-automated-sdlc/drupal-sdlc/*/templates/mcp.json 2>/dev/null | tail -1
```

Copy that file to `.mcp.json` in the current project root.

If the template is also not found, create `.mcp.json` with this exact content:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Tell the user:
> `.mcp.json` has been created. Commit this file to git so your team members get it too.

---

### Check 3 — enableAllProjectMcpServers

Run:
```bash
cat .claude/settings.local.json 2>/dev/null | grep -c enableAllProjectMcpServers || echo "0"
```

**If found (count > 0):** Pass. Continue.

**If NOT found (0):** Fix it automatically.

Read `.claude/settings.local.json` if it exists, add `"enableAllProjectMcpServers": true` to the JSON, and write it back.

If the file doesn't exist, create `.claude/settings.local.json` with:
```json
{
  "enableAllProjectMcpServers": true
}
```

Tell the user:
> `enableAllProjectMcpServers` has been enabled. This allows the project's MCP servers to start automatically. **Restart Claude Code** for this to take effect.

---

### Check 4 — Playwright MCP connectivity

Check if the Playwright MCP tools are available by trying to use browser_navigate or browser_take_screenshot.

If Playwright MCP tools are not available, tell the user:
> Playwright MCP is not connected. This usually means:
> 1. `.mcp.json` was just created — restart Claude Code to load it
> 2. `enableAllProjectMcpServers` was just set — restart Claude Code
> 3. npx can't find the package — run `npx @playwright/mcp@latest --help` in your terminal to test

---

### Check 5 — GitHub MCP connectivity

If `drupal-claude.yml` exists, read `github.owner` and `github.repo` from it. Then use the GitHub MCP to list pull requests on that repository.

If `drupal-claude.yml` does not exist yet, try listing repos for the authenticated user via GitHub MCP.

**If the GitHub MCP returns results:** Pass. Continue.

**If it fails:** Tell the user:
> GitHub MCP is not connected. Most likely causes:
> 1. `GITHUB_TOKEN` is not set (see Check 1)
> 2. `.mcp.json` was just created — restart Claude Code
> 3. The token has insufficient scopes — needs `repo` + `read:org`

---

### Check 6 — Atlassian MCP connectivity

Use Atlassian MCP `atlassianUserInfo` to verify the connection is active.

**If it returns user info:** Pass. Continue.

**If it fails:** Tell the user:
> Atlassian MCP is not connected. To fix:
> 1. Open Claude Code Settings (gear icon)
> 2. Go to MCP Servers
> 3. Find Atlassian → click Disconnect → Connect again
> 4. Authorize with your Axelerant Atlassian account

---

### Check 7 — drupal-claude.yml

Run:
```bash
cat drupal-claude.yml 2>/dev/null | head -1 || echo "NOT FOUND"
```

**If found:** Pass. Continue.

**If NOT FOUND:** Auto-create from plugin template.

```bash
ls ~/.claude/plugins/cache/axelerant-automated-sdlc/drupal-sdlc/*/templates/drupal-claude.yml 2>/dev/null | tail -1
```

Copy that template to `drupal-claude.yml` in the project root.

Tell the user:
> `drupal-claude.yml` has been created. Open it and fill in every field — especially `project.name`, `local_dev.base_url`, `github.owner`, `github.repo`, and `jira.project_key`.

---

### Check 8 — CLAUDE.md

Run:
```bash
cat CLAUDE.md 2>/dev/null | head -1 || echo "NOT FOUND"
```

**If found:** Pass. Continue.

**If NOT FOUND:** Auto-create from plugin template.

```bash
ls ~/.claude/plugins/cache/axelerant-automated-sdlc/drupal-sdlc/*/templates/CLAUDE.md 2>/dev/null | tail -1
```

Copy that template to `CLAUDE.md` in the project root.

Tell the user:
> `CLAUDE.md` has been created. Fill in the project name, Jira key, and GitHub repo at the top.

---

### Summary

After all checks, show this table:

| Check | Status | Action needed |
|-------|--------|---------------|
| GITHUB_TOKEN | pass/fail | — or fix command |
| .mcp.json | pass/fail | — or created |
| enableAllProjectMcpServers | pass/fail | — or created |
| Playwright MCP | pass/fail | — or restart needed |
| GitHub MCP | pass/fail | — or fix |
| Atlassian MCP | pass/fail | — or reconnect |
| drupal-claude.yml | pass/fail | — or created, needs filling |
| CLAUDE.md | pass/fail | — or created, needs filling |

**If any checks created files or changed settings:**
> Some files were created or settings were changed. **Restart Claude Code** and re-run `onboarding` to verify everything is connected.

**If all checks pass:**
> Everything is set up. You can now run:
> ```
> work on jira ticket KEY-X
> ```

---

## Hard Rules

- Run every check in order — do not skip
- Do not run `work on jira ticket` or any other skill until all checks pass
- When auto-creating config files, always copy from the plugin template — never generate content from scratch
- Never fill in drupal-claude.yml values yourself — always ask the user to fill them in
- Always tell the user to commit `.mcp.json` and `drupal-claude.yml` to git so team members get them
