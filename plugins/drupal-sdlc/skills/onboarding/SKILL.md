---
name: onboarding
description: "Run post-install setup checks for the drupal-sdlc plugin. Verifies GITHUB_PERSONAL_ACCESS_TOKEN, MCP connectivity, Node.js version, and project config files. Auto-creates drupal-claude.yml and CLAUDE.md if missing. Use after installing the plugin on a new project, or when MCPs show as failed."
argument-hint: ""
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
- ✅ **Check N — Name**: one-line result
- ❌ **Check N — Name**: what's wrong + exact fix

At the end, show a summary table and tell the user what to do next.

---

### Check 1 — GITHUB_PERSONAL_ACCESS_TOKEN

Run:
```bash
if [ -n "$GITHUB_PERSONAL_ACCESS_TOKEN" ]; then echo "SET"; else echo "MISSING"; fi
```

**If SET:** ✅ Continue.

**If MISSING:** ❌ Tell the user:

> `GITHUB_PERSONAL_ACCESS_TOKEN` is not set. The GitHub MCP server reads this specific variable — it is different from `GITHUB_TOKEN` (used by the `gh` CLI).
>
> Run this to fix it (replace with your actual token):
> ```shell
> echo 'export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_your_token_here' >> ~/.zshrc
> source ~/.zshrc
> ```
> Then **restart Claude Code completely**. Re-run `/onboarding` after restarting.

Stop here if this check fails — the remaining checks will also fail until this is set.

---

### Check 2 — Node.js version

Run:
```bash
node --version 2>/dev/null || echo "NOT FOUND"
```

**If v20.x.x or higher:** ✅ Continue.

**If below v20 or NOT FOUND:** ❌ Tell the user:

> Node.js 20+ is required for the Playwright MCP server. Install it from nodejs.org, then restart Claude Code.

---

### Check 3 — GitHub MCP connectivity

Read `drupal-claude.yml` to get `github.owner` and `github.repo`. Then use the GitHub MCP to list pull requests on that repository.

If `drupal-claude.yml` does not exist yet, skip to Check 5 — you need the config file before you can test GitHub MCP against the right repo.

**If the GitHub MCP returns results (even "no open PRs"):** ✅ Continue.

**If it fails with "Not Found" or authentication error:** ❌ Tell the user:

> GitHub MCP cannot access the repository. Most likely causes:
> 1. `GITHUB_PERSONAL_ACCESS_TOKEN` is set but the session hasn't restarted since it was added — restart Claude Code
> 2. The token has insufficient scopes — it needs `repo` + `read:org`
> 3. The `.mcp.json` in this project is missing or malformed — check that `.mcp.json` exists at the project root

---

### Check 4 — Atlassian MCP connectivity

Use Atlassian MCP `atlassianUserInfo` to verify the connection is active.

**If it returns user info:** ✅ Continue.

**If it fails:** ❌ Tell the user:

> Atlassian MCP is not connected. To fix:
> 1. Open Claude Code Settings (gear icon)
> 2. Go to MCP Servers
> 3. Find Atlassian → click Disconnect → Connect again
> 4. Authorize with your Axelerant Atlassian account (yourname@axelerant.com)

---

### Check 5 — drupal-claude.yml

Run:
```bash
cat drupal-claude.yml 2>/dev/null | head -1 || echo "NOT FOUND"
```

**If found:** ✅ Continue.

**If NOT FOUND:** Auto-create it from the plugin template.

Run:
```bash
ls ~/.claude/plugins/cache/axelerant-automated-sdlc/drupal-sdlc/*/templates/drupal-claude.yml 2>/dev/null | tail -1
```

Copy the template from that path to `drupal-claude.yml` in the current directory.

Then tell the user:
> `drupal-claude.yml` has been created from the plugin template. Open it now and fill in every field marked with a comment — especially:
> - `project.name`
> - `local_dev.base_url`
> - `local_dev.quality_command`
> - `github.owner` and `github.repo`
> - `jira.project_key`

---

### Check 6 — CLAUDE.md

Run:
```bash
cat CLAUDE.md 2>/dev/null | head -1 || echo "NOT FOUND"
```

**If found:** ✅ Continue.

**If NOT FOUND:** Auto-create it from the plugin template.

Run:
```bash
ls ~/.claude/plugins/cache/axelerant-automated-sdlc/drupal-sdlc/*/templates/CLAUDE.md 2>/dev/null | tail -1
```

Copy the template from that path to `CLAUDE.md` in the current directory.

Then tell the user:
> `CLAUDE.md` has been created from the plugin template. Open it and fill in the project name, Jira project key, and GitHub repo details at the top.

---

### Check 7 — Plugin version

Run:
```bash
cat ~/.claude/plugins/installed_plugins.json 2>/dev/null | grep -A5 'drupal-sdlc'
```

Tell the user:
- The installed version and git SHA
- The current latest version is 1.1.0
- If the installed version is 1.0.0 or the SHA is `4b822f5`, the plugin is outdated

If outdated, tell the user:
> Your plugin is outdated. Run these commands to reinstall the latest version:
> ```
> /plugin install drupal-sdlc@Pulkit0111/axelerant-automated-sdlc
> /reload-plugins
> ```

---

### Summary

After all checks, show this table:

| Check | Status | Action needed |
|-------|--------|---------------|
| GITHUB_PERSONAL_ACCESS_TOKEN | ✅/❌ | — or fix |
| Node.js 20+ | ✅/❌ | — or fix |
| GitHub MCP | ✅/❌ | — or fix |
| Atlassian MCP | ✅/❌ | — or fix |
| drupal-claude.yml | ✅/❌ | — or created |
| CLAUDE.md | ✅/❌ | — or created |
| Plugin version | ✅/❌ | — or reinstall |

**If all checks pass:**
> Everything is set up correctly. You can now run:
> ```
> work on jira ticket KEY-X
> ```

**If any checks failed:**
> Fix the items marked ❌ above, then re-run `/onboarding` to verify.

---

## Hard Rules

- Run every check in order — do not skip
- Do not run `work on jira ticket` or any other skill until all checks pass
- When auto-creating config files, always copy from the plugin template — never generate content from scratch
- Never fill in drupal-claude.yml values yourself — always ask the user to fill them in
