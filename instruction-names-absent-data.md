---
name: instruction-names-absent-data
description: "an agent body naming an input the harness never emits (\"check the packet's topic list\") reads as the agent ignoring it, so rubric/wording/model-tier fixes all look right and all miss; grep the producing code for the field, and render the missing input inside the consumer's budgeted body with an explicit empty marker"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 7b7bd319-48a2-48ff-8c72-f10ca829d2dc
  modified: 2026-10-31T13:58:24.796Z
---

Every noun phrase in an agent body that names an input — "the packet's topic
list", "the list above", "your task file" — is a claim about what the harness
emits, and it is verified by grepping the producing code for the field or by
serving one real input and looking at it. An instruction can be written true of
an intended design and never checked against the renderer that has to supply
it.

**Why:** an instruction whose data is absent fails as *apparent misjudgment*,
which makes every natural hypothesis wrong and expensive. The agent looks like
it ignored the instruction, so the fixes that suggest themselves are about the
agent — the rubric drifted, the model tier is too low, the wording needs
sharpening. Each is testable, each will seem to help on a small sample, and
none addresses the cause. The instruction was never violated: nothing was there
to consult, and the agent cannot report a gap it has no way to perceive. So
when agent output degrades in a way that looks like judgment, check the inputs
before the prompt.

ghmem's triage agent carried "check the packet's current topic list for a
near-duplicate" while its packet renderer emitted a header, an item id and a
rendered thread and nothing else. Two concurrent workers then minted 20 new
topic names for 22 items, including two synonyms for the same ground in one
run, beside a third topic that already covered it.

This is "green is not evidence" pointed at a prompt rather than a test — a
check is evidence only when some state of the world makes it fail
(`craft:test-discipline`) — and
the sibling of a guard's ALLOW case (`craft:test-discipline`, "Assert the
invocation contract"): there a guard was
checked against a synthetic command instead of a real one, here an instruction
against a design instead of a renderer. Same root — the rule was never run
against the artifact it talks about.

**How to apply:** supplying the missing input, put it where the consumer's own
accounting already reaches. A list rendered into a free header rather than the
budgeted body leaves every downstream size calculation short by its length, and
a truncation computed to fit a budget then overshoots it. State the empty case
explicitly (`topics: (none)`) rather than omitting the line, so the reader can
never confuse "nothing to consult" with "the input forgot" — that confusion is
the original failure returning in a form that looks like data.
