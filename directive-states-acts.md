---
name: directive-states-acts
description: "writing an agent-facing or script-emitted directive, or a code comment: what to cut from it, and what \"remove narration\" cuts"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8cd21851-d241-49bf-afaf-b4baf0c4f311
  modified: 2026-08-10T05:07:13.981Z
---

When authoring text a script emits *for an agent to act on* — a probe directive,
a hook's `additionalContext`, an error message that instructs — include only the
acts the reader performs. Cut everything else:

- **Mechanism.** What the receiving system does with the file, which hook
  consumes it, the mtime freshness rule behind an ordering, a gitlink being
  staged. State the ordering; never the rule that motivates it.
- **What an earlier step already settled.** If the skill body sequences memory
  before the probe, the probe must not re-order them.
- **A meta-description of the constraint.** "no second file, no trigger" was a
  note about what the text must *avoid*; it got literalized into the text.
- **Rationale for why the mode exists.** The reader is executing, not reviewing
  the design.
- **Prohibitions naming a thing the reader could not otherwise reach.** See
  below — this one has its own test.

Keep one clause of reassurance where the absence of a visible result would
otherwise read as failure ("the commit that lands these changes carries the
memory") — that prevents an invented recovery. Make it true of every flow the
mode covers: the first draft said "this turn's commit", false wherever the
commit arrives in a later user turn.

**Withhold, don't forbid.** When the only route to an unwanted action is an
identifier — a filename, a flag, an endpoint — the reader has no other source
for, withhold it. Naming `.claude/gitlore-commit-memory` to say "do not write
it" hands a fresh agent the exact string that triggers the behaviour the mode
exists to prevent. The prohibition also *introduces the concept*, and an
introduced concept gets reasoned about. Withholding is not politeness — it is
what makes the action unreachable. The counter-argument ("a reader who already
knows the protocol needs it forbidden, not omitted") assumes a rare reader and
pays for it with the common one; maintainer docs like a project `CLAUDE.md` are
not in a foreign project's context. Ask who else could supply the identifier.
Nobody → say nothing. Only when the reader demonstrably already has it (it
appeared earlier in the same context) is a prohibition worth its cost, and then
forbid by role rather than by name.

**Code comments take the same cut, against a different bar: what a maintainer
needs in order to change the code safely.** Where a repo keeps its design
history in a changelog, comments drift into retelling it — the defect that
motivated the shape, what an earlier draft said, a clause defending the choice
against an alternative. The rule, the constraint and the contract stay; the
story, the incident and the self-commentary belong to the changelog, which is
where a reader who wants them is already headed. Rationale a maintainer needs
in order not to break the code is not narration: "the arrays expand this way
because bash 3.2" stays. "Remove narration" is the instruction that makes the
cut.

**One change, four artifacts, one telling.** A change that touches code, tests,
a changelog entry and a design doc invites writing the incident into all four,
and each copy reads as reasonable in isolation. The changelog gets the narrative
— it is the write-time record; the design doc gets the standing rule in present
tense; the code and test comments get the constraint alone. Watch for a
signature phrase recurring across them: "load-bearing", "the shape of", "by
construction" carry real meaning the first time and become verbal tics by the
fourth, at which point they read as emphasis substituting for content. Grep the
diff for your own favourites before committing.

**Why:** handoff's `with-commit` memory directive shipped as five paragraphs
explaining gitlore's pre-commit hook, gitlink staging, mtime freshness, and an
explicit "do NOT write `.claude/gitlore-commit-memory`". User: "This is still
saying much more than the agent needs to know. That kind of prose leads to
annoying verbosity when the agent narrates the superfluous checks." Mechanism a
reader cannot act on does not sit inert — it gets verified, narrated back, and
worked around. Guidance restating a condition the reader has already satisfied
asks a question whose only possible answer is narration. On the prohibition: the
reader is a fresh agent in an arbitrary project with no other source for that
filename, so naming it — even to forbid it — was the only way it could cause the
split the mode exists to prevent.

**How to apply:** draft the directive, then delete every sentence that is not an
act. Ask what a reader would *do* with each remaining line; if the answer is
"check it" or "explain it back", cut it. Test the negatives on the whole word
(`[[ "$output" != *"trigger"* ]]`) and pair each with a positive over the same
fixture per [[green-is-not-evidence]]. When a rule is an ordering between the
*caller's own steps*, it belongs where the steps are described: a prohibition
must be front-loaded, a procedure defers to its trigger. The split of judgement
from mechanism is the harness/agent boundary; stating a rule positively in the
step where it acts rather than as a prohibition is stated in gitlore's
`memory-writing` skill.
