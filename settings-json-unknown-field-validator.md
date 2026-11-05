---
name: settings-json-unknown-field-validator
description: Claude Code's settings.json validator is a hand-rolled Zod schema that rejects unknown top-level keys despite the published JSON Schema declaring additionalProperties:true. For plugin config use the pluginConfigs[plugin@marketplace].options.{key} slot, not a custom top-level key.
metadata:
  type: feedback
  originSessionId: 47e452e1-587d-4dce-85b8-249840d26793
---

The Edit/Write tools run a strict validation pass against
`.claude/settings.json` after the write. That pass rejects unknown
top-level keys with "Unrecognized field: X" and a Zod error payload
(`"code": "unrecognized_keys"`) — even though the published JSON
Schema at `json.schemastore.org/claude-code-settings.json` declares
`additionalProperties: true` at the root, and even though the schema
dump in the error message itself has `additionalProperties: {}`.
The runtime Zod schema is canonical; the published JSON Schema is
documentation that lags it. (Confirmed by Anthropic issues #3481,
#5886, #28742.)

**Why:** This rules out the "just stick a top-level key in
settings.json" approach for plugin config. The validator is strict at
the top level but lax inside nested objects, so the fix is to nest.

**How to apply:**

1. **For plugin config**, write to
   `pluginConfigs["<plugin>@<marketplace>"].options.<key>`. This is
   the schema-recognized slot for plugin-specific config. Values
   constrained to `string | number | boolean | string[]`. Edit/Write
   work cleanly through this path (validator passes). Example for the
   gitmoji plugin: `pluginConfigs["gitmoji@ddaanet"].options.enabled`.

2. **Known top-level keys** (`hooks`, `permissions`, `enabledPlugins`,
   `pluginConfigs`, etc.): Edit/Write work as expected. Field list is
   at <https://code.claude.com/docs/en/settings>, but treat the page
   as best-effort — even documented fields have been seen rejected
   when the internal schema lags (issue #28742).

**jq syntax for the @ in the marketplace name:** use bracket form
because `@` is not allowed in unquoted jq identifiers:

```
jq '.pluginConfigs["gitmoji@ddaanet"].options.enabled = true' settings.json
```

When constructing object literals in jq, the same rule applies:
`{pluginConfigs: {"gitmoji@ddaanet": {options: {enabled: true}}}}`.
