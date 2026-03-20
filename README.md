# Axelerant Automated SDLC for Drupal

AI-native SDLC automation plugin for Claude Code.
Spec-first, test-first Drupal development with Jira integration.

## Quick Install

```
/plugin marketplace add Pulkit0111/axelerant-automated-sdlc
/plugin install drupal-sdlc@axelerant-sdlc --scope project
```

## Setup

1. Copy `drupal-claude.yml` template to your project root:
   ```
   cp <plugin-path>/templates/drupal-claude.yml ./drupal-claude.yml
   ```
2. Fill in your project values (GitHub owner/repo, Jira project key, DDEV project name, etc.)
3. Copy `CLAUDE.md` template to your project root and customize it
4. Copy CI templates to `.github/workflows/`:
   ```
   cp <plugin-path>/templates/ci.yml .github/workflows/ci.yml
   cp <plugin-path>/templates/jira-transition.yml .github/workflows/jira-transition.yml
   ```
5. Set up GitHub secrets for Jira integration:
   - `JIRA_USER_EMAIL` — your Jira account email
   - `JIRA_API_TOKEN` — your Jira API token
   - `JIRA_BASE_URL` — your Jira instance URL (e.g. `https://mycompany.atlassian.net`)
6. Start working: `work on jira ticket TDP-XX`

## Skills Included

| Skill | Description |
|---|---|
| `/spec-writer` | Generate structured specs with numbered ACs and test mapping |
| `/config-builder` | Generate Drupal config YAML from plain English |
| `/module-scaffolder` | Scaffold custom Drupal modules |
| `/test-writer` | Generate Playwright and PHPUnit tests |
| `/validate` | Run quality + config round-trip + test validation |
| `/code-quality-fixer` | Fix linter violations (PHPStan, PHPCS, twigcs) |
| `/pr-creator` | Create pull requests with full context |
| `/pr-reviewer` | Review PRs with security checklist and AC coverage |
| `/work-on-jira-ticket` | Full SDLC automation: spec → build → test → PR → review |
| `/hotfix` | Urgent fix bypass workflow (skips spec for speed) |

## How It Works

1. **Trigger:** `work on jira ticket TDP-XX`
2. **Read:** Fetches ticket details from Jira via MCP
3. **Spec:** Generates implementation plan with Drupal-specific constructs
4. **Approve:** Human checkpoint — you review and approve the spec
5. **Build:** Generates config YAML, scaffolds modules, writes code
6. **Validate:** Runs quality checks, config round-trip, visual verification
7. **Test:** Generates and runs Playwright + PHPUnit tests
8. **PR:** Creates PR with full context, test results, risk level
9. **Review:** Automated code review with security checklist
10. **Done:** GitHub Action transitions Jira ticket on merge

## What's Included

- **10 skills** — full SDLC coverage from spec to review
- **3 rules** — Drupal config YAML, custom module, and API integration patterns
- **Hooks** — protected file guard + PHP syntax check on save
- **Permissions** — pre-configured allowlist for Drupal/Jira/GitHub tools
- **CI templates** — GitHub Actions for testing and Jira automation
- **MCP config** — Playwright browser automation

## Requirements

- Claude Code CLI
- DDEV (or compatible local development tool)
- Drupal 10 or 11
- GitHub repository
- Jira project (optional but recommended)
- Node.js 20+ (for Playwright tests)

## Documentation

- [Getting Started](docs/getting-started.md)
- [How It Works](docs/how-it-works.md)
- [Skills Reference](docs/skills-reference.md)
- [Configuration Reference](docs/configuration-reference.md)
- [CI Setup](docs/ci-setup.md)
- [Customization](docs/customization.md)

## License

MIT
