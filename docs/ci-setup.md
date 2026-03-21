# CI Setup

## Overview

The plugin provides two GitHub Actions workflow templates:

1. **ci.yml** — runs on every PR: installs Drupal, runs quality checks, config validation, and Playwright tests
2. **jira-transition.yml** — runs when a PR is merged: automatically transitions the Jira ticket to Done

## Installing the Workflows

Copy the templates to your project:

```bash
mkdir -p .github/workflows
cp <plugin-path>/templates/ci.yml .github/workflows/ci.yml
cp <plugin-path>/templates/jira-transition.yml .github/workflows/jira-transition.yml
```

## CI Workflow (ci.yml)

### What It Does

1. **Checkout** — clones the PR branch
2. **DDEV Setup** — installs and starts DDEV
3. **Composer Install** — installs PHP dependencies (cached by `composer.lock`)
4. **Security Audit** — runs `composer audit --no-dev` to check for known vulnerabilities
5. **Drupal Install** — fresh install with config import:
   - `site:install standard`
   - Sets site UUID from `config/sync/system.site.yml`
   - Deletes shortcut_set (avoids config import conflict)
   - `updatedb -y`
   - `config:import -y` (with retry on failure)
   - `cache:rebuild`
6. **Config Validation** — `config:status` must show "No differences"
7. **Config Round-Trip** — `config:export -y` + `git diff --exit-code config/sync/` to verify committed config matches exported config
8. **Quality Checks** — runs the project's quality command (GrumPHP, PHPStan, PHPCS, etc.)
9. **Playwright Tests** — installs browsers (cached), runs tests
10. **Artifact Upload** — uploads test results on success or failure (14-day retention)

### Configuration Required

Edit these values in `ci.yml`:

```yaml
env:
  DDEV_PROJECT_NAME: my-project         # Your DDEV project name
  BASE_URL: https://my-project.ddev.site # Your DDEV site URL
```

### Caching

The workflow caches:
- **Composer dependencies** — keyed by `composer.lock`
- **Playwright browsers** — keyed by `package-lock.json`
- **npm modules** — via `setup-node` cache

## Jira Transition Workflow (jira-transition.yml)

### What It Does

1. **Extract ticket key** — parses the branch name for a Jira ticket key (e.g. `feat/TDP-12-something` → `TDP-12`)
2. **Get transitions** — queries Jira for available transitions on the ticket
3. **Transition to Done** — moves the ticket to Done status
4. **Retry on failure** — waits 5 seconds and retries once if the first attempt fails
5. **Post completion comment on Jira** — posts a "Completed" comment with PR link, merge date, and who merged it
6. **Post failure comment on PR** — if transition still fails, posts a comment on the GitHub PR so the team knows to transition manually

### Required GitHub Secrets

| Secret | Description | Example |
|---|---|---|
| `JIRA_USER_EMAIL` | Jira account email | `developer@company.com` |
| `JIRA_API_TOKEN` | Jira API token | Generate at https://id.atlassian.com/manage-profile/security/api-tokens |
| `JIRA_BASE_URL` | Jira instance URL | `https://mycompany.atlassian.net` |

### Branch Name Convention

The workflow extracts the Jira ticket key from the branch name using this pattern: any uppercase letters followed by a dash and digits (e.g. `TDP-12`, `PROJ-5`).

Supported branch name formats:
- `feat/TDP-12-description`
- `hotfix/TDP-12-description`
- `fix/TDP-12-description`
- `TDP-12-description`

If no ticket key is found, the workflow warns but does not fail.

## Troubleshooting

### Config import fails in CI

The CI workflow retries `config:import` once. If it still fails:
- Check that `updatedb` runs before `config:import`
- Verify the site UUID is set correctly
- Check for module dependency issues

### Playwright tests fail in CI but pass locally

- Ensure `BASE_URL` matches your DDEV project URL
- Check that the health check step passes (site is responding)
- Review uploaded artifacts for screenshots and traces

### Jira transition fails

- Verify GitHub secrets are set correctly
- Check that the Jira API token hasn't expired
- Ensure the ticket isn't already in Done status
- Check that the "Done" transition exists in your Jira workflow
