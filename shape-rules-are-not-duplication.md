---
name: shape-rules-are-not-duplication
description: "which layer a validation rule belongs in: ask what vocabulary the rule needs"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ca597732-d159-40c8-adc3-c5e8c88c1ec2
  modified: 2026-08-11T19:04:11.199Z
---

When deciding whether an outer guard should reject something the inner
layer already rejects, the question is not "is this duplication?" but
**"does the rule need knowledge only the inner layer has?"**

If it does, leave it inside — two copies of a domain rule drift. If it
does not, it is a fact about the request's *shape*, and the guard is
where it belongs.

The call went backwards on ghmem's allowlist hook. Asked whether an
empty heredoc note should be legal, the argument was that the hook
should stay content-agnostic: it is a grammar, and which verdicts
require a note lives in `record_verdict` with both halves tested. My
human partner overruled it — an empty heredoc is malformed either way,
because a verdict that takes a note has none and one that takes no
note should not have opened a heredoc at all.

That rule never has to know which verdict is in play, so it was never
the domain rule. The "don't duplicate" instinct had bundled two
different rules under one label and defended the wrong one.

**Why:** the layer that owns a rule is set by what the rule needs to
decide, not by which layer happens to catch it first. Get that wrong
and a malformed command is accepted by the guard, then accepted or
rejected inconsistently downstream depending on an unrelated field —
here, `no-signal` would have taken an empty note silently while every
other verdict errored.

**How to apply:** before declining to add a check on duplication
grounds, write the rule out and see which layer's vocabulary it needs.
"The heredoc must carry something" needs none of the verdict
vocabulary; "a topic verdict needs a note" needs all of it. Put each
where its vocabulary lives, and enforce the shape rule at *both*
boundaries when a caller can reach the inner layer another way — the
CLI got the same empty-note rejection so a non-hooked caller cannot do
what the hook forbids. See [[no-code-for-impossible-cases]] and
[[dont-bake-in-guarantees]] for the neighbouring judgment calls.
