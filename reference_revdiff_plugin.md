---
name: revdiff-plugin
description: revdiff plugin (umputun/revdiff marketplace) installed for in-session diff review via /revdiff:revdiff
metadata: 
  node_type: memory
  type: reference
  originSessionId: 7bed82ba-de53-4fd5-9e88-49823dea7260
---

User has the `revdiff@revdiff` plugin enabled in this project
(`.claude/settings.json`). Marketplace: `umputun/revdiff`. CLI:
`/usr/local/bin/revdiff`.

`/revdiff:revdiff` opens a TUI overlay for annotating diffs/files;
empty output on exit = review approved with no changes. The skill
classifies annotations into code-change directives (apply) vs
explanation requests (answer first, loop until accepted).

**How to apply:** When the user asks to review staged changes
mid-task, invoke `/revdiff:revdiff` rather than dumping a diff. See
[[tmux-cc-popup]] for the popup-vs-new-window caveat — in CC mode
the launcher's `display-popup` strategy fails silently and a
`tmux new-window` fallback is required.
