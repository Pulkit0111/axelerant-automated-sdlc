# Configuration Reference

All project-specific configuration lives in `drupal-claude.yml` at the project root. This is the only file teams need to edit to configure the plugin.

## Full Schema

```yaml
project:
  name: ""                    # Project name (used in commit messages and PR references)
  drupal_version: 11          # Drupal major version (10 or 11)
  php_version: "8.3"          # PHP version

local_dev:
  tool: ddev                  # Local dev tool: ddev | lando | docksal
  base_url: ""                # Local site URL (e.g. "https://my-project.ddev.site")
  drush_prefix: "ddev drush"  # Command prefix for drush (e.g. "ddev drush", "lando drush")
  quality_command: ""         # Full command to run quality checks (e.g. "ddev composer run-script quality")

entity_types:
  content_types: []           # List of content type machine names
  taxonomies: []              # List of taxonomy vocabulary machine names
  paragraph_types: []         # List of paragraph type machine names

protected_files: []           # File paths that should never be modified without approval

high_risk_modules: []         # Module names that need extra review when modified

api_integrations: []          # External API names this project connects to

config_splits: []             # Config split environment names (e.g. [dev, stage, prod])

playwright:
  test_dir: "tests/playwright"       # Path to Playwright test directory
  base_url_env_var: "BASE_URL"       # Environment variable name for base URL

github:
  owner: ""                   # GitHub organization or username
  repo: ""                    # GitHub repository name

jira:
  cloud_id: ""                # Jira cloud ID (e.g. "mycompany.atlassian.net")
  project_key: ""             # Jira project key (e.g. "TDP", "PROJ")
  board_url: ""               # Full Jira board URL
```

## Field-by-Field Documentation

### project

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Used in branch names and PR context |
| `drupal_version` | Yes | Affects module scaffolding templates (^10 vs ^11) |
| `php_version` | No | Informational, used in CI setup |

### local_dev

| Field | Required | Description |
|---|---|---|
| `tool` | Yes | Determines command prefixes and environment assumptions |
| `base_url` | Yes | Used for Playwright tests and visual verification |
| `drush_prefix` | Yes | Prepended to all drush commands (e.g. config:import, config:export) |
| `quality_command` | Yes | Full command run after every code change |

### entity_types

| Field | Required | Description |
|---|---|---|
| `content_types` | No | Helps spec-writer and config-builder know what exists |
| `taxonomies` | No | Same as above |
| `paragraph_types` | No | Enables paragraph-aware config generation |

### protected_files

List of file paths that the pre-tool hook will block from editing. Default includes `settings.php` and `settings.ddev.php`. The hook also blocks `.install` files and `config_split.config_split.*` files regardless of this list.

### high_risk_modules

Module names listed here trigger a "High" risk score in pr-reviewer when modified.

### api_integrations

List of external API names. Helps spec-writer and module-scaffolder generate appropriate integration patterns.

### config_splits

Environment names for config splits. When set, config-builder will mention split-aware placement to the user.

### playwright

| Field | Required | Description |
|---|---|---|
| `test_dir` | Yes | Path to Playwright test directory relative to project root |
| `base_url_env_var` | Yes | Environment variable name used in Playwright config and CI |

### github

| Field | Required | Description |
|---|---|---|
| `owner` | Yes | Used by pr-creator and pr-reviewer to create PRs via GitHub MCP |
| `repo` | Yes | Same as above |

### jira

| Field | Required | Description |
|---|---|---|
| `cloud_id` | Yes | Used by all Jira MCP calls for authentication |
| `project_key` | Yes | Used in branch naming, PR titles, and Jira queries |
| `board_url` | No | Informational, included in spec comments |
