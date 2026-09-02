---
name: parse-dont-regex-structured-formats
description: "use a real parser for YAML/JSON/TOML/HTML, never a hand-rolled regex even for \"just one field\"; regex is for prose token extraction"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2ab4326a-2ac4-4c90-9dd8-4da55efdf552
  modified: 2026-08-11T19:03:57.699Z
---

For a structured format with a grammar — YAML, JSON, TOML, HTML — reach
for the proper parsing library, never a hand-rolled regex, even for
"just one field". My human partner's framing: "don't use regex where
you should really use a parser. You know, zalgo and all."

**Why:** a regex over a grammar breaks on the inputs the grammar
allows but the pattern didn't anticipate (quoting, escapes, multiline,
nesting) — and it fails silently, matching the wrong thing rather than
erroring. A real parser encodes the grammar once and rejects malformed
input loudly.

**How to apply:** if the proper parser needs a dependency, add the
dependency — in ghmem C1, reading a one-line `description:` from doc
frontmatter, `msgspec.yaml.decode` needed PyYAML, so the right move
was adding `pyyaml`, not hand-parsing the field. The exception is
genuine token extraction from freeform prose (e.g. pulling bare `#N`
issue references out of markdown body text): there is no grammar to
parse, so a regex — validated against real data — is the right tool.
Structural splitting of an unambiguous delimiter (the `---`
frontmatter fence) by explicit line handling is fine too; it is the
*format's content* that must go through the parser. Validate the regex
against real data before shipping it.
