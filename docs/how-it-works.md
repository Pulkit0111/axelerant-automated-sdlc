# How It Works

## Architecture

The plugin is a single Claude Code plugin (`drupal-sdlc`) containing:

```
drupal-sdlc/
├── skills/          10 slash commands for SDLC automation
├── rules/           3 context rules for Drupal patterns
├── hooks/           Pre/Post tool hooks for safety
├── settings.json    Permissions allowlist
└── .mcp.json        Playwright MCP server config
```

## Component Overview

### Skills (Slash Commands)

Skills are the primary interface. Each skill is a markdown file that defines a workflow with instructions, context injection, and guardrails.

| Skill | Role in Workflow |
|---|---|
| `work-on-jira-ticket` | Master orchestrator — calls all other skills in sequence |
| `spec-writer` | Generates implementation spec with Drupal-specific constructs |
| `config-builder` | Generates Drupal config YAML (fields, content types, displays) |
| `module-scaffolder` | Scaffolds custom modules (services, forms, routing) |
| `test-writer` | Generates Playwright and PHPUnit tests |
| `validate` | Runs quality checks, config round-trip, visual verification |
| `code-quality-fixer` | Fixes linter violations |
| `pr-creator` | Creates GitHub PRs with full context |
| `pr-reviewer` | Reviews PRs with security and AC coverage checks |
| `hotfix` | Streamlined urgent fix workflow |

### Rules

Rules provide contextual patterns that activate based on file paths:

- **config-yaml-patterns** — activates when editing `config/**/*.yml`
- **custom-module-patterns** — activates when editing `web/modules/custom/**`
- **api-integration-patterns** — activates when editing `web/modules/custom/**`

### Hooks

Two hook types protect code quality:

1. **PreToolUse (Edit|Write)** — blocks writes to protected files (settings.php, .install, config_split)
2. **PostToolUse (Write|Edit)** — runs `php -l` syntax check on PHP files after every edit

### MCP Servers

The plugin configures Playwright MCP for browser automation (visual verification, screenshot capture).

GitHub and Atlassian MCP servers must be configured separately as they require authentication tokens.

## Workflow Diagram

```
Jira Ticket
    │
    ▼
┌──────────────┐
│ spec-writer   │  ◄── Human approval gate
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ config-builder│  and/or  ┌─────────────────┐
│              │           │ module-scaffolder │
└──────┬───────┘           └────────┬─────────┘
       │                            │
       └────────────┬───────────────┘
                    │
                    ▼
              ┌──────────┐
              │ validate  │  quality + config + screenshots
              └─────┬────┘
                    │
                    ▼
              ┌──────────┐
              │test-writer│  Playwright + PHPUnit
              └─────┬────┘
                    │
                    ▼
              ┌──────────┐
              │pr-creator │  GitHub MCP
              └─────┬────┘
                    │
                    ▼
              ┌──────────┐
              │pr-reviewer│  security + AC coverage
              └─────┬────┘
                    │
                    ▼
              Human Review  ◄── Human merge gate
                    │
                    ▼
              GitHub Action → Jira Done
```

## How Context Flows

1. **drupal-claude.yml** — single source of truth for all project-specific values
2. Skills read `drupal-claude.yml` at invocation time via shell expansion (`!`cat drupal-claude.yml``)
3. Skills also inject live state: current branch, changed files, existing content types, enabled modules
4. Rules activate automatically based on file path patterns
5. Hooks fire automatically on every file edit/write

## Human Checkpoints

The workflow has two mandatory human checkpoints:

1. **After spec generation** — you must review and explicitly approve the implementation plan
2. **After PR creation** — you must review and merge the PR yourself

The plugin will NEVER merge a PR or proceed past these checkpoints without your explicit approval.

## Error Handling

Every skill has an "On Failure" section. The general pattern:
1. Post error context as a Jira comment (so the ticket has a record)
2. Present the error to the user
3. Stop and wait for guidance — never silently proceed
