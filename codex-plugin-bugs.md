---
name: codex-plugin-bugs
description: "`/codex:review` refused or \"no actionable defects\"/\"no regressions\" after a skim: use adversarial-review --base with a focus file as its own argv (template leaves `$(cat …)` unquoted); companion and `/codex:cancel` need sandbox off (`mkdir …/jobs` ENOENT, EROFS, `error building bubblewrap command`); `--model` takes full slugs from `~/.codex/models_cache.json` (short `astra`/`sol` fail \"not supported when using Codex with a ChatGPT account\"), no `--effort`; \"codex-code-mode-host is missing\""
metadata:
  node_type: memory
  type: reference
  originSessionId: 82e309ea-799a-45ff-9c2d-026fd7e696cd
  modified: 2026-10-27T22:08:03.597Z
---

`openai-codex` plugin (companion `codex-companion.mjs`, plugin 1.0.6):

- **`/codex:review` takes no positional argument** — it is read as focus
  text and refused. It reviews a diff only (working tree or `--base <ref>`),
  so a committed doc on a clean tree needs `--base HEAD~1`.
- **Native review skims.** On docs it reports "no actionable defects"
  without checking claims; on a 36-file code range it ran seven commands and
  said "no regressions" where a focused pass found four defects (2026-10-27).
  Treat a fast native "clean" as unreviewed.
- **Use `adversarial-review --base <ref>` with a focus file**: the artifact,
  what is settled, sources to verify against, probes, finding format. A
  one-line focus underuses it — my human partner's correction. The slash
  template wraps all args in one double-quoted string, leaving `$(cat …)`
  unquoted (split, globbed); run the companion with flags and
  `"$(cat file)"` as separate argv — with more than one argument it uses
  them as-is (`normalizeArgv`).
- **`--model <slug>`** works on both reviews, but only with a full ID
  (`gpt-6-astra`, `gpt-6.1-sol`, …). A short name (`--model astra`, `sol`)
  fails at the first turn with `The 'astra' model is not supported when
  using Codex with a ChatGPT account.` (`model_not_found`): the message
  blames the account, and re-logging in or upgrading the plan changes
  nothing. List the IDs with
  `jq -r '.models[] | .slug // .id' ~/.codex/models_cache.json`; the default
  is the `model =` line in `~/.codex/config.toml`. When a short name matches
  several IDs (`gpt-6.1-sol`, `gpt-6-sol`, `gpt-5.6-sol`), pick the newest
  and say so (seen 2026-10-29). No `--effort`: effort comes from
  `~/.codex/config.toml`. `/codex:adversarial-review` is user-invoked only.
- **Run the companion unsandboxed.** Sandboxed it fails on job state under
  `~/.claude/plugins/data/codex-openai-codex/` (`ENOENT … mkdir …/jobs`,
  `EROFS`), or Codex's own bubblewrap cannot start (`Read-only file system`)
  and the review exits 0 with a body saying it saw no diff. The
  `/codex:cancel` command runs its `!` shell sandboxed and fails on the same
  job-state error (seen 2026-10-16). Unsandboxed,
  `$TMPDIR` is unset: let the background task log capture stdout.
- Codex's sandbox cannot create scratch repos or `mktemp` in `/tmp`, so its
  git findings and bats runs come back inferred — verify before folding.
- `/codex:setup` `ready: true` does not check `codex-code-mode-host`; when
  missing, review "completes" with "Unable to verify … is missing".

Related: the `sandbox-lies:sandbox-lies` skill.
