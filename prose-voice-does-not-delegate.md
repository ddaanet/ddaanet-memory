---
name: prose-voice-does-not-delegate
description: "an underspecified \"uncram\" brief gets applied as find-and-replace; NOT evidence that prose degrades down-tier, and \"flat\" was my own unconfirmed verdict"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3f455518-26f6-4b60-9d14-22f0e25d6f36
  modified: 2026-07-25T07:41:03.596Z
---

Editing comments and docstrings for *style* delegates badly when the
brief states a goal without a method.
Given a brief saying "split em-dash pile-ups" and "unpack telegraphic
fragments", subagents systematically converted every single em-dash and
every explanatory colon into a full stop, producing limp replacements
("Not its own table — nested JSON inside…" became "This is not its own
table. It is nested JSON inside…"). A corrective wave was worse: it left
files whose exact regressions had been quoted to it untouched,
over-reverted others back to zero, and raced my own edits until I
stopped it.

**Why:** the brief said *what* to do but never *how* to unpack a run-on
paragraph, and an underspecified style rule affords only find-and-
replace. The judgement it actually requires — *is this edit clearer, or
just longer?* — was never briefed, so it was never applied. Terse voice
reads as "crammed" to an agent told to uncram.

**Two limits on this evidence, both load-bearing.** "Limp" and "flat"
were *my own* verdicts on the replacements in that session; my human
partner never confirmed a regression, so this is a self-assessment, not
a measured quality drop. And the cause was brief specification, not
model tier — nothing here says a cheaper model would have done worse
against a *well*-specified brief. Do not cite this memory as a prior
that prose degrades down-tier; it does not support that claim.

**How to apply:** brief the *how*, not just the what — a worked
before/after example of an unpacking, and an explicit "keep the
em-dash, split the ideas" beats any amount of restating the goal. Where
that cannot be written down, delegate the mechanical, verifiable half
(semicolon key-chains into bullet lists; splitting a genuinely
multi-idea paragraph) and keep the rest. Structural work with a
machine-checkable exit condition delegates fine — the `raw_nodes.py` split with
"`check_line_limits.sh` must exit 0" came back correct on the first try.
Whatever comes back, read the full diff before reporting it as done;
"agent reported success" is not verification. See
[[mutate-code-to-test-the-tests]] for the same evidence-over-assertion
instinct, and [[honest-line-count-caps]] for why the uncramming was
allowed to push a file over the cap in the first place.
