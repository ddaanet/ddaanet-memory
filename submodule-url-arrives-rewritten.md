---
name: submodule-url-arrives-rewritten
description: "matching a submodule's `remote.origin.url` against a marker you wrote into `.gitmodules`: git absolutizes a relative url on the way in, so the literal is only ever visible to a `.gitmodules` read; and an entry with no `url` at all cannot be initialized"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6e0477fe-1f1f-42e9-bd70-94fc717a7601
  modified: 2026-09-02T22:12:25.033Z
---

A submodule entry's `url` travels through git before it reaches the
submodule's own `remote.origin.url`, so a marker value planted in
`.gitmodules` is not the string that arrives.

Verified on git 2.47.3:

- **No `url` key at all** — `git submodule init`, `update --init` and
  `clone --recurse-submodules` all die with `fatal: No url found for
  submodule path '<path>' in .gitmodules`. Everything else is indifferent:
  `git status`, `git submodule status`, the gitlink itself, and a
  `git config --file .gitmodules submodule.<name>.path` lookup all work.
  So an entry that must stay initializable needs *some* url, which is the
  only thing a placeholder value buys.
- **A relative url is absolutized.** `submodule init` and `submodule sync`
  resolve `./.git/marker` against the superproject's location and write
  `<parent-abs-path>/.git/marker`. An equality test against the registered
  spelling can only match a `.gitmodules` read; against
  `remote.origin.url` it is dead code. Match the basename, or read
  `.gitmodules`.
- **A placeholder does not rescue a clone.** `submodule update --init` on
  one fails with `repository '<abs>/.git/marker' does not exist` — the
  same net outcome as no url, a different message.
- **In that state origin exists**, so the repair path has to
  `remote set-url`; `remote add` fails.

**How to apply:** before writing a comparison against a sentinel you
planted in a config another tool owns, run the tool and read back what it
stored. A check against the spelling you wrote passes review, never
matches, and silently inverts the branch it guards — gitlore's
`create-remote.sh` read a synced placeholder as a real remote already
wired and refused to create one.
