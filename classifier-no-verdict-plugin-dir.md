---
name: classifier-no-verdict-plugin-dir
description: "`The server-side auto mode classifier gave no verdict (it skipped this action)` on a Write inside a plugin checkout loaded with `--plugin-dir .` (a dogfood shim): every file there is sensitive, and a retry fails the same way — write through a Python script in `$TMPDIR`, never a heredoc; a delegate hits it too"
metadata:
  type: feedback
---

In a session started with `--plugin-dir .` — which is what a dogfood
`.bin/claude` shim does — the auto-mode classifier treats every file in the
plugin checkout as sensitive. A `Write` there returns "gave no verdict (it
skipped this action)", says the failure is hard, and a retry fails the same way.
Seen in sandbox-lies on 2026-09-28 (CC 2.1.283) on reports under `plans/`,
from the main session and from three delegates.

**Why:** my human partner gave the cause, and the working route, after all
four writes in a deliverable review failed.

**How to apply:**

- Write the content from a Python script saved under `$TMPDIR` (the Write
  tool reaches `$TMPDIR` fine) and run with `python3`. My human partner named
  the script route and ruled out a `cat > file <<EOF` heredoc.
- A delegate in such a repo that must write a file needs this in its prompt.
  Otherwise it returns its report inline, and the main session has to merge it.
- The route covers this no-verdict gap only. A refusal that names a reason is a
  denial, and no route around it is sanctioned.

Related: [[classifier-denied-self-config]], which covers refusals that name a
reason. This one names none.
