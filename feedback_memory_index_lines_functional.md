---
name: feedback-memory-index-lines-functional
description: "MEMORY.md index lines aren't disposable summaries — they're what the agent matches against for recall AND what it holds ambiently in context; don't mechanically shorten them to hit a byte target"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 5c5b043b-8460-4841-a796-694a07a15915
  modified: 2026-07-24T16:12:36.440Z
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
Separately, the byte-budget check turned out to be advisory — it never
refused composition even at 103%, so there wasn't a real forcing function
for the trim in the first place. Caught by the user; the index was
restored to full original text (see [[feedback_verify_hook_threshold_claims]]
for the companion lesson about the unverified-threshold half of this
mistake).

**How to apply:** don't mechanically compress index lines to satisfy a
byte-budget warning. If a line carries genuinely redundant prose
(transitions, "why it matters" scaffolding already implied by the title),
cutting that is fine; cutting a specific command, flag, error string, or
file path is not — even if it's the thing making the line long. Check
whether a size constraint is a hard gate or merely advisory before treating
it as a forcing function for content loss.
