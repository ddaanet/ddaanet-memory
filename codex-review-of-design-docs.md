---
name: codex-review-of-design-docs
description: "`/codex:review <path>` refused / \"no actionable defects\" on docs: use adversarial-review --base with a focus file; companion needs sandbox off (`mkdir …/jobs` ENOENT); \"codex-code-mode-host is missing\" despite setup ready"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 82e309ea-799a-45ff-9c2d-026fd7e696cd
  modified: 2026-09-15T18:01:03.846Z
---

Running the `openai-codex` Claude Code plugin (companion `codex-companion.mjs`,
plugin 1.0.6, codex-cli 0.154.0) against a plan or outline:

- **`/codex:review <path>` fails.** A positional argument is parsed as focus
  text, which native review refuses. It reviews only a diff (working tree, or
  `--base <ref>`), so a committed doc on a clean tree needs `--base HEAD~1`.
- **Native review is useless on docs.** It is a code-defect reviewer: on a
  docs-only diff it reports "no executable code … no actionable defects"
  without checking any claim.
- **Use `adversarial-review --base <ref> "<focus>"`** with a detailed focus:
  which file is the artifact, what is settled and not to relitigate, which
  source files to verify citations against, the probe list, and the finding
  format (severity, citation, failure scenario, fix). Keep the focus in a file
  and pass `"$(cat file)"`. A bare one-line focus underuses the reviewer — my
  human partner's correction.
- **The companion must run with the sandbox disabled.** It writes job state
  under `~/.claude/plugins/data/codex-openai-codex/`, which the sandbox denies
  (`ENOENT … mkdir …/jobs`). Unsandboxed, `$TMPDIR` is unset, so don't
  redirect output there — let the background task log capture stdout.
- Codex's own sandbox cannot create scratch repos, so its git-behaviour
  findings come back inferred, not reproduced — verify them before folding.
- `/codex:setup` reporting `ready: true` does not check the
  `codex-code-mode-host` helper beside the `codex` binary. When it is missing,
  review "completes" with "Unable to verify … codex-code-mode-host is missing".

Related: [[sandbox-effects]].
