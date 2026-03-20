# Customization

## Adding Project-Level Skills

You can add project-specific skills alongside the plugin skills. Create them in your project's `.claude/skills/` directory:

```
your-project/
├── .claude/
│   └── skills/
│       └── my-custom-skill/
│           └── SKILL.md
├── drupal-claude.yml
└── ...
```

Project-level skills won't conflict with plugin skills — they have different namespaces.

## Extending Hooks

### Adding custom pre-tool hooks

Add hooks to your project's `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'your-custom-hook-script'"
          }
        ]
      }
    ]
  }
}
```

Project hooks run alongside plugin hooks — both will fire.

### Adding more protected files

Add paths to the `protected_files` list in `drupal-claude.yml`:

```yaml
protected_files:
  - web/sites/default/settings.php
  - web/sites/default/settings.ddev.php
  - web/sites/default/services.yml
  - config/sync/system.site.yml
```

Note: The hook also always blocks `.install` files and `config_split.config_split.*` regardless of this list.

## Overriding Permissions

Add additional permissions to your project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(your-custom-command *)"
    ]
  }
}
```

Project permissions are merged with plugin permissions.

## Customizing the Quality Command

Change the quality command in `drupal-claude.yml`:

```yaml
local_dev:
  quality_command: "ddev exec vendor/bin/grumphp run"
```

All skills read this value dynamically — no need to update individual skill files.

## Using a Different Local Dev Tool

The plugin supports any local dev tool. Change the prefix:

```yaml
local_dev:
  tool: lando
  base_url: "https://my-project.lndo.site"
  drush_prefix: "lando drush"
  quality_command: "lando composer run-script quality"
```

## Adding Custom Rules

Create rule files in your project's `.claude/rules/` directory:

```markdown
---
paths:
  - "web/themes/custom/**"
---

# Custom Theme Patterns

Your custom rules for theme development...
```

## Customizing CI

The CI templates are starting points. Common customizations:

### Adding deployment steps

```yaml
      - name: Deploy to staging
        if: github.event.pull_request.merged == true
        run: |
          # Your deployment script
```

### Adding custom quality tools

```yaml
      - name: Custom security scan
        run: ddev exec vendor/bin/security-checker check
```

### Changing Playwright configuration

Update the Playwright test command in `ci.yml`:

```yaml
      - name: Run Playwright tests
        run: npx playwright test --config=tests/playwright/playwright.config.ts --workers=2
```

## Disabling Skills

You cannot selectively disable individual plugin skills, but you can:
1. Not use them — simply don't invoke them
2. Create project-level skills with the same names to override behavior
3. Adjust the `work-on-jira-ticket` workflow by creating a project-level version

## Customizing the Spec Format

Create a project-level `spec-writer` skill that overrides the plugin version, with your preferred output format. Place it in `.claude/skills/spec-writer/SKILL.md`.
