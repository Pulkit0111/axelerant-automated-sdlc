---
paths:
  - "web/modules/custom/**"
---

# Custom Module Patterns

## Module Types

- API integration: API wrapper + business logic service + ConfigFormBase + entity hooks/event subscribers
- Feature: Form alters, Views plugins, controllers, theme hooks
- Migration: Migrate plugins, Drush commands, source/process/destination plugins
- Utility: Single-purpose modules (batch operations, field widgets, filters)

## .info.yml Standard

name: '{Human Name}'
type: module
description: '{Description}'
package: Custom
core_version_requirement: ^10 || ^11

## Service Registration (.services.yml)

- API wrapper: `arguments: ['@config.factory', '@http_client', '@logger.factory']`
- Event subscribers: `tags: [{ name: event_subscriber }]`
- Drush commands: `tags: [{ name: drush.command }]`

## Admin Routing

Recommended path pattern: `/admin/config/services/{module}/`

Route definition:
  path: '/admin/config/services/{module}'
  defaults:
    _controller: '\Drupal\system\Controller\SystemController::systemAdminMenuBlockPage'
    _title: '{Human Name}'
  requirements:
    _permission: 'access administration pages'

## Config Forms

- Extend `ConfigFormBase`, implement `getEditableConfigNames()`
- Config name pattern: `{module}.settings`
- Config schema in `config/schema/{module}.schema.yml`
- Default values in `config/install/{module}.settings.yml`

## Common Hooks (by typical frequency)

- hook_form_alter — most common, for form modifications
- hook_theme — template registration
- hook_views_query_alter — Views query customization
- hook_user_login — triggered on login, often used for API sync
- hook_entity_insert / hook_entity_update — entity lifecycle
