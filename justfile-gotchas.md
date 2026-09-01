---
name: justfile-gotchas
description: "writing a just recipe: body lines share one indent (column-0 breaks it), `{{ }}` is parsed inside a recipe body's `#` comments too (`Unknown start of token`, or SILENT substitution when the name is in scope; file-level comments are inert), and shell quotes round `{{arg}}` protect nothing since the recipe's own shell evaluates the caller's `$(…)`; use `quote()`"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a14e1c39-21ac-4c1a-b7d6-544b48807ded
  modified: 2026-09-01T15:20:41.313Z
---

Three things about just that bite while writing a recipe, none of which a
shell-shaped intuition predicts.

**Interpolation is raw text, so shell quotes around it protect nothing.**
`{{ ... }}` is textual substitution into the recipe body before any shell
sees it — never a shell-level parameter. By the time the shell parses the
line, the argument's own `$(...)`, backticks and quote characters are part
of the source text. A recipe written as `--arg c "{{command}}"` runs
whatever substitution the caller passed and hands the downstream program
something the caller never typed, silently and with exit 0. Pass any
argument that is *data* rather than an identifier through just's `quote()`
function, unquoted at the shell level, since `quote()` emits its own single
quotes: `--arg c {{ quote(command) }}`. Verified against just 1.46. A
recipe whose argument reaches a program that interprets it — a commit
message, a shell command, a path — is where this bites, and the form is
common across ddaanet justfiles (a `test msg='...'` recipe echoing the
message into a file, a `sync name=''` recipe testing `[ -n "..." ]`).

**Every body line of a shebang recipe must share a common leading indent**,
which just strips before executing. Lines starting at column 0 — a pasted
Python or awk heredoc body, say — are not exempt; they break the recipe
parse because they violate the shared-indent rule. Either indent the whole
block uniformly or move it to a standalone script the recipe calls.

**`{{ ... }}` is parsed inside a recipe's `#` comment lines too**, so a
comment written to *describe* interpolation syntax is itself a parse error
(`Unknown start of token`, pointing at the comment). What the braces hold
decides how loudly it fails (just 1.46): literal `{{ ... }}` gives that token
error, an undefined name gives ``error: Variable `x` not defined``, and a name
already in scope — a parameter of the very recipe the comment sits in — is
substituted **silently**. So a comment explaining why an argument needs
`quote()` acquires the caller's argument and says something the author never
wrote, while the recipe parses and runs. Only a recipe *body*'s comments are
scanned: a file-level comment above the recipe is inert, literal braces
included. Describe interpolation in prose, without the braces, wherever the
comment sits.

**Why:** the last two both surface as a token error on a line that looks
inert, which invites blaming the embedded language. Concluding that
embedded Python in a just recipe was the problem and rewriting it in jq
drew a correction from my human partner: jq is fine, the real issue was
indentation. Blaming the tool for the author's own mistake wastes an
iteration and misleads future edits. The first one never surfaces as an
error at all.

**How to apply:** when a shebang recipe fails to parse with a token error,
check the indent of every body line and the braces in every comment before
touching the embedded script. When a recipe takes an argument, decide
whether it is an identifier or data, and wrap the data in `quote()`.
