# Getting Started

## Prerequisites

- **Claude Code CLI** installed and authenticated
- **DDEV** installed and working
- **Drupal 10 or 11** project with Composer
- **GitHub repository** for your project
- **Node.js 20+** for Playwright tests
- **Jira project** (optional but recommended for full workflow)

## Step 1: Install the Plugin

```
/plugin marketplace add Pulkit0111/axelerant-automated-sdlc
/plugin install drupal-sdlc@axelerant-sdlc --scope project
```

## Step 2: Configure Your Project

### Copy the configuration template

Copy `drupal-claude.yml` from the plugin's templates directory to your project root:

```bash
cp <plugin-path>/templates/drupal-claude.yml ./drupal-claude.yml
```

### Fill in your values

Edit `drupal-claude.yml` and fill in all the TODO fields:

```yaml
project:
  name: "my-drupal-project"
  drupal_version: 11
  php_version: "8.3"

local_dev:
  tool: ddev
  base_url: "https://my-project.ddev.site"
  drush_prefix: "ddev drush"
  quality_command: "ddev composer run-script quality"

entity_types:
  content_types: [article, page]
  taxonomies: [tags]
  paragraph_types: []

protected_files:
  - web/sites/default/settings.php
  - web/sites/default/settings.ddev.php

github:
  owner: "your-org"
  repo: "your-repo"

jira:
  cloud_id: "your-company.atlassian.net"
  project_key: "PROJ"
  board_url: "https://your-company.atlassian.net/jira/software/projects/PROJ/boards/1"
```

### Copy the CLAUDE.md template

```bash
cp <plugin-path>/templates/CLAUDE.md ./CLAUDE.md
```

Customize the project overview, Jira details, and GitHub details sections.

## Step 3: Set Up CI

### Copy workflow files

```bash
mkdir -p .github/workflows
cp <plugin-path>/templates/ci.yml .github/workflows/ci.yml
cp <plugin-path>/templates/jira-transition.yml .github/workflows/jira-transition.yml
```

### Update CI configuration

Edit `.github/workflows/ci.yml`:
- Set `DDEV_PROJECT_NAME` to your DDEV project name
- Set `BASE_URL` to your DDEV site URL

### Set GitHub Secrets

In your GitHub repository settings, add these secrets:
- `JIRA_USER_EMAIL` — your Jira account email
- `JIRA_API_TOKEN` — generate at https://id.atlassian.com/manage-profile/security/api-tokens
- `JIRA_BASE_URL` — your Jira instance URL (e.g. `https://mycompany.atlassian.net`)

## Step 4: Set Up MCP Servers

The plugin requires three MCP servers. **Playwright is included automatically.** GitHub and Atlassian must be configured by each user since they require personal auth tokens.

### Playwright MCP (included)

No action needed — the plugin's `.mcp.json` configures this automatically.

### GitHub MCP (required for PRs)

Add to your Claude Code MCP settings (or project `.mcp.json`):

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<your-github-token>"
      }
    }
  }
}
```

**To get a GitHub token:**
1. Go to https://github.com/settings/tokens
2. Generate a new token (classic) with `repo` scope
3. Set it as the `GITHUB_PERSONAL_ACCESS_TOKEN` value above

Alternatively, set `GITHUB_TOKEN_JIRA` as an environment variable and reference it:
```json
"GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN_JIRA}"
```

### Atlassian MCP (required for Jira)

Atlassian MCP is a Claude-managed integration. To connect it:
1. Open Claude Code settings
2. Go to MCP Servers → Add Server
3. Search for "Atlassian" and connect your Atlassian account
4. Authorize access to your Jira instance

Once connected, the plugin's skills can read tickets, post comments, and transition issues automatically.

### Verify MCP connections

Test each MCP server works:
- **GitHub:** Ask Claude "list open PRs on {owner}/{repo}" — should return results
- **Atlassian:** Ask Claude "read Jira ticket {KEY}-1" — should return the ticket
- **Playwright:** Run `/drupal-sdlc:validate` — visual verification step should work

## Step 5: Verify Installation

1. Start a new Claude Code session in your project
2. Type `/` and verify you see the plugin skills (e.g. `drupal-sdlc:validate`)
3. Run `/drupal-sdlc:validate` to verify the quality checks work
4. Try: `work on jira ticket PROJ-1` to test the full workflow

## Verification Checklist

- [ ] `drupal-claude.yml` exists in project root with all values filled
- [ ] `CLAUDE.md` exists in project root
- [ ] CI workflows are in `.github/workflows/`
- [ ] GitHub secrets are configured
- [ ] Plugin skills appear in `/` menu
- [ ] `/drupal-sdlc:validate` runs successfully
- [ ] Atlassian MCP can read Jira tickets
- [ ] GitHub MCP can create PRs
