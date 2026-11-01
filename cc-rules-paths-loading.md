---
name: cc-rules-paths-loading
description: "a `.claude/rules/*.md` with `paths:` loads once, on the first Read, Edit or Bash `cat` of a matching file, subagents included; the session that created `.claude/rules/` sees nothing on a matching Read, so test from a fresh `claude -p`; `**` matches zero directories"
metadata:
  node_type: memory
  type: reference
  originSessionId: 392c60ed-2774-428d-b574-5498ba472f2e
  modified: 2026-10-31T14:57:27.772Z
---

A `.claude/rules/*.md` file with `paths:` frontmatter (a YAML block list of
globs relative to the repository root) arrives as a system-reminder carrying
its contents, once, at the first access to a matching file. Observed on
Claude Code 2.1.316:

- A Read of a matching file loads it, in the main session and in a
  general-purpose subagent; a Read of a non-matching file loads nothing. A
  dispatch brief need not name the rules file, provided the subagent's edit
  starts with a read of a matching file.
- An Edit of a matching file loads it too. Write was not probed.
- A Bash `cat` or `sed` of a matching file can load it, but detection is
  uneven across compound commands; rely on a Read when testing.
- `**` matches zero directories: `scripts/**/*.sh` matches
  `scripts/write-guard.sh`.

A session does not see a rules directory created after it launched. In the
session that created `.claude/rules/`, a Read of a matching file loaded
nothing, while a subagent of that same session, spawned before the file
existed, loaded it on its first matching access. The same main session did
load a rules file on a matching Edit after a `/compact`; whether the
compaction made the difference is unprobed. So test a new rules file from a
fresh `claude -p` or a subagent: in the session that wrote it, silence reads
as a broken `paths:` glob.

`claude plugin validate --strict` does not read `.claude/rules/` in a plugin
repository, so rules files add no validator finding.
