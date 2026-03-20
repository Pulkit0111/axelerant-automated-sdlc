# Config Reference — Drupal Field Type Templates

Use these as templates when generating new config. Replace placeholder names with actual field names.

## Field Storage Templates

### text_long

```yaml
langcode: en
status: true
dependencies:
  module:
    - node
    - text
id: node.field_example_text
field_name: field_example_text
entity_type: node
type: text_long
settings: {  }
module: text
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### datetime

```yaml
langcode: en
status: true
dependencies:
  module:
    - datetime
    - paragraphs
id: paragraph.field_example_date
field_name: field_example_date
entity_type: paragraph
type: datetime
settings:
  datetime_type: date
module: datetime
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### entity_reference

```yaml
langcode: en
status: true
dependencies:
  module:
    - node
    - user
id: node.field_example_user
field_name: field_example_user
entity_type: node
type: entity_reference
settings:
  target_type: user
module: core
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### entity_reference_revisions (paragraphs)

```yaml
langcode: en
status: true
dependencies:
  module:
    - entity_reference_revisions
    - node
    - paragraphs
id: node.field_example_paragraph
field_name: field_example_paragraph
entity_type: node
type: entity_reference_revisions
settings:
  target_type: paragraph
module: entity_reference_revisions
locked: false
cardinality: -1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### boolean

```yaml
langcode: en
status: true
dependencies:
  module:
    - user
id: user.field_example_flag
field_name: field_example_flag
entity_type: user
type: boolean
settings: {  }
module: core
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### list_string

```yaml
langcode: en
status: true
dependencies:
  module:
    - node
    - options
id: node.field_example_type
field_name: field_example_type
entity_type: node
type: list_string
settings:
  allowed_values:
    -
      value: option_one
      label: 'Option One'
    -
      value: option_two
      label: 'Option Two'
  allowed_values_function: ''
module: options
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

### link

```yaml
langcode: en
status: true
dependencies:
  module:
    - link
    - node
id: node.field_example_link
field_name: field_example_link
entity_type: node
type: link
settings: {  }
module: link
locked: false
cardinality: -1
translatable: true
indexes: {  }
persist_with_no_fields: false
custom_storage: false
```

---

## Field Instance Templates

### text_long instance

```yaml
langcode: en
status: true
dependencies:
  config:
    - field.storage.node.field_example_text
    - node.type.article
  module:
    - text
id: node.article.field_example_text
field_name: field_example_text
entity_type: node
bundle: article
label: 'Example Text'
description: ''
required: false
translatable: false
default_value: {  }
default_value_callback: ''
settings: {  }
field_type: text_long
```

### entity_reference instance (with handler)

```yaml
langcode: en
status: true
dependencies:
  config:
    - field.storage.node.field_example_user
    - node.type.article
id: node.article.field_example_user
field_name: field_example_user
entity_type: node
bundle: article
label: 'User reference'
description: ''
required: false
translatable: false
default_value: {  }
default_value_callback: ''
settings:
  handler: 'default:user'
  handler_settings:
    target_bundles: null
    sort:
      field: _none
      direction: ASC
    auto_create: false
    filter:
      type: _none
    include_anonymous: false
field_type: entity_reference
```

### entity_reference_revisions instance (paragraph)

```yaml
langcode: en
status: true
dependencies:
  config:
    - field.storage.node.field_example_paragraph
    - node.type.article
  module:
    - entity_reference_revisions
id: node.article.field_example_paragraph
field_name: field_example_paragraph
entity_type: node
bundle: article
label: 'Example Paragraph'
description: ''
required: false
translatable: false
default_value: {  }
default_value_callback: ''
settings:
  handler: 'default:paragraph'
  handler_settings:
    negate: 0
    target_bundles:
      example_type: example_type
    target_bundles_drag_drop:
      example_type:
        enabled: true
        weight: 0
field_type: entity_reference_revisions
```

### boolean instance (with default value and labels)

```yaml
langcode: en
status: true
dependencies:
  config:
    - field.storage.user.field_example_flag
  module:
    - user
id: user.user.field_example_flag
field_name: field_example_flag
entity_type: user
bundle: user
label: 'Example flag'
description: ''
required: false
translatable: false
default_value:
  -
    value: 0
default_value_callback: ''
settings:
  on_label: 'On'
  off_label: 'Off'
field_type: boolean
```

---

## Form Display Entry Template

Add to the `content` map in `core.entity_form_display.*.yml`:

```yaml
field_name_here:
  type: string_textfield
  weight: 10
  region: content
  settings: {  }
  third_party_settings: {  }
```

Common widget settings:
- `string_textfield`: `settings: { size: 60, placeholder: '' }`
- `text_textarea`: `settings: { rows: 5, placeholder: '' }`
- `entity_reference_autocomplete`: `settings: { match_operator: CONTAINS, match_limit: 10, size: 60, placeholder: '' }`
- `options_select`: `settings: { }`
- `boolean_checkbox`: `settings: { display_label: true }`
- `datetime_default`: `settings: { }`

---

## View Display Entry Template

Add to the `content` map in `core.entity_view_display.*.yml`:

```yaml
field_name_here:
  type: string
  label: above
  settings: {  }
  third_party_settings: {  }
  weight: 10
  region: content
```

Label options: `above`, `inline`, `hidden`
