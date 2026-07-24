---
name: justfile shebang recipes need consistent indentation
description: In just, shebang recipes strip only the common leading indent — mixed or column-0 lines cause parse errors. Do not conclude the language choice is the problem.
type: feedback
originSessionId: 19f8e87d-f395-4b41-8b8b-9db2e9276a03
---
In a just `#!/usr/bin/env bash` (or any shebang) recipe, every line in the
body must share a common leading indent; just strips that shared indent
before executing. Lines starting at column 0 (e.g. a pasted Python
heredoc body) are not exempt — they break the recipe parse because they
violate the "common indent" rule.

**Why:** I concluded that embedded Python in a just recipe was the
problem and rewrote it in jq. The user corrected: jq is fine, but the
real issue was indentation, not the language. Blaming the tool when the
mistake was mine wastes an iteration and misleads future edits.

**How to apply:** If a just shebang recipe fails to parse with a token
error on an embedded script line, first check that every body line is
indented at least as much as the recipe's shebang. If you need a
multi-line Python/awk/etc. block, either indent the whole block
uniformly or move it to a standalone script file and call it from the
recipe.
