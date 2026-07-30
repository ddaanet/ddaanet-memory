---
name: memory-index-lines-functional
description: "index lines serve agent-matching AND ambient awareness; don't mechanically shorten them to hit a byte target, especially an advisory one"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-07-24T19:27:42.885Z
---

`MEMORY.md`'s one-line entries do two jobs at once: they're what a
matching/recall pass (or the model itself scanning the always-loaded index)
pattern-matches against to decide a fact is relevant, and — since the whole
index loads into every session — they're the actual ambient knowledge
available without ever opening the linked topic file. Shortening a line to
hit a byte target strips exactly the distinctive tokens (specific commands,
flags, error-message substrings, file paths, version numbers) both
functions depend on.

**Why:** after routing 44 local facts into a newly mounted gitlore tier,
`MEMORY.md` came in over gitlore's byte budget. I mechanically shortened
~50 index lines to fit, and when asked whether that was safe, defended it
as "no content loss — the full text still lives in the topic file." That
missed the point: the topic file being intact doesn't help if the
shortened index line no longer contains what a future recall pass would
match on, or what I'd recognize ambiently without going to read the file.
Caught by the user; the index was restored to full original text (see
[[verify-at-source]] for the companion lesson about the
unverified-threshold half of this mistake).

**Two budgets, and only one of them is advisory.** gitlore's own check
(`GITLORE_INDEX_BUDGET_BYTES`, 25600, warning at 80%) reports and never
refuses composition — it stayed quiet-but-unheeded at 103%. **Claude Code's
own loader is a hard cap:** past roughly 25 KB it silently truncates the
always-loaded index and says so once (`MEMORY.md is 25.8KB (limit: 24.4KB)
— Only part of it was loaded`). Observed 2026-07-24 at 26754 B: the last
six entries never reached the session.

**Truncation takes the tail, and composition puts project lines there.** The
composed root index is `[tier blocks, manifest order] → [project bare-path
lines]`, so an overflowing index loses the *project-local* memories first —
the ones most specific to the repo you are in. That ordering is correct on
its own terms (broad facts first, specific ones override), which is exactly
why the cap must be treated as a failure mode to stay under rather than a
degraded mode to tolerate.

So: the forcing function is real, but it is CC's cap, not gitlore's warning.
Stay under it by **merging overlapping facts and retiring reconstructable
ones**, never by stripping trigger tokens from surviving lines.

**How to apply:** don't mechanically compress index lines to satisfy a
byte-budget warning. If a line carries genuinely redundant prose
(transitions, "why it matters" scaffolding already implied by the title),
cutting that is fine; cutting a specific command, flag, error string, or
file path is not — even if it's the thing making the line long. Check
whether a size constraint is a hard gate or merely advisory before treating
it as a forcing function for content loss.
