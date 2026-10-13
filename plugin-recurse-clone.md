---
name: plugin-recurse-clone
description: "`/plugin install` clones `--recurse-submodules`, so a relative submodule url → `not a valid repository name` and install aborts; `--plugin-dir` does not recurse, so dogfooding can't catch it; `Failed to clone repository` / `Failed to recurse into submodule path 'memory'` for an installer without access: `\"source\": \"github\"` clones over SSH with no https fallback, a private nested tier aborts, `submodule.<name>.update = none` skips it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fba96ba4-0800-400a-b860-509be142f52d
  modified: 2026-10-13T01:17:20.335Z
---

`/plugin install <name>@<scope>` clones the plugin's GitHub repo with `git clone --recurse-submodules`. Any submodule committed in that repo is therefore cloned into every installer's `~/.claude/plugins/cache/...`.

**A relative or local-only submodule url breaks install.** gitlore committed `.gitmodules` with `url = ./.git/gitlore-placeholder` (the local-only sentinel) plus a gitlink. On install, git resolved the relative url against the remote → `git@github.com:ddaanet/gitlore.git/.git/gitlore-placeholder` → "not a valid repository name" → install aborts. Discovered Plan 04 Step 6 (2026-05-23) — the inner-loop dogfood could not catch it because `--plugin-dir` does not recurse-clone.

Requirements for a distributable plugin repo with a self-hosted submodule:
- `.gitmodules` url must be **absolute** and use a real scheme (`https://`, `git@…`, `ssh://`). Use **https** so anonymous installers can read a public repo without SSH keys.
- The submodule's recorded gitlink SHA must be **pushed and fetchable** on that remote.
- For install to succeed for everyone, the submodule remote must be **public** (a private submodule → install works only for those with access).

**Three layers abort an install for someone without the owner's GitHub access** (verified against CC 2.1.296, `claude plugin install` in a scratch `CLAUDE_CONFIG_DIR`):

1. A marketplace entry with `"source": "github"` clones the plugin repo over SSH, and the install step has no https fallback (`marketplace add` does: `SSH clone failed, retrying with HTTPS`). A `{"source": "url", "url": "https://…git"}` entry clones over https. A `git-subdir` entry with an `owner/repo` shorthand `url` clones over SSH too; the full https url does not.
2. An SSH submodule url → `Permission denied (publickey)` → `Failed to clone repository`.
3. The recurse is `--recursive`: a private repo nested inside a public submodule (a gitlore tier under `memory/`) aborts it too, with `Failed to recurse into submodule path 'memory'`.

`submodule.<name>.update = none` in the plugin repo's `.gitmodules` clears 2 and 3: git prints `Skipping submodule` and the install succeeds with the directory empty.

To simulate a keyless installer, put a stub `ssh` that exits 255 first on `PATH`. A scratch `HOME` is not enough: `ssh` reads keys from the passwd home.

Fix shipped: created public `ddaanet/gitlore-memory`, pushed the submodule, set `.gitmodules` url to `https://github.com/ddaanet/gitlore-memory.git`. Verified by `git clone --recurse-submodules git@github.com:ddaanet/gitlore.git` succeeding. Regression guard: `tests/plugin_distribution.bats` asserts the submodule url is an absolute scheme. The local-only placeholder mechanism still applies to **user** installs (`/gitlore:install` in a target repo) — it's only the plugin repo's *own* self-dogfood submodule that needs a real remote. See [[cc-agent-discovery]].
