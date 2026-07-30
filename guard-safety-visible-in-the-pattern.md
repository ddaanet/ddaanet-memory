---
name: guard-safety-visible-in-the-pattern
description: "a containment guard's accepted-input class should be narrow enough that its safety is visible in the pattern, not a proof about quoting — even when a differential test shows the wide class is safe"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ee821d18-029f-4b49-8a7a-56008312ab1a
  modified: 2026-07-26T13:48:37.593Z
---

When a guard admits user-chosen input, narrow the class to what is
actually needed rather than to what is provably safe. "Any run of
non-whitespace minus the quote" and "letters, digits and underscores"
were equally safe for ghmem's heredoc delimiter; only the second is
safe *at a glance*.

**Why:** my human partner asked whether "any run of non-whitespace" was
really supported by shell syntax and whether it could break containment.
A differential harness over 36 exotic delimiters (`$(…)`, backticks,
`>`, `|`, `&&`, `*`, `\`, `"`) found no divergence between the hook's
parse and bash's — the delimiter word is single-quoted, bash expands
nothing in it, and quote removal returns it verbatim. So the wide class
was safe. But the entire guarantee rested on one excluded character,
and reading it required a paragraph of reasoning about quoting rather
than a look at the pattern. Nobody needs a `$` in a delimiter.

**How to apply:** ask what the writer actually needs, not what you can
prove inert. When the answer is "a word", accept a word. Run the
differential test anyway — it is what tells you whether you are
narrowing out of caution or out of necessity, and the answer belongs in
the comment. See [[guardrail-must-permit-real-commands]] and
[[tests-pass-for-the-wrong-reason]] — two of the tests for that class
were passing because a *different* rule caught them, and only survived
mutation once each body closed with exactly what its opener named.
