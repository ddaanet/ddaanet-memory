---
description: "Cross-project facts shared by all ddaanet repositories"
---

# ddaanet tier index

Portable facts that hold across every ddaanet project — tooling behaviour,
Claude Code mechanics, shell and git gotchas. Project-specific facts stay in
the project's own memory store.

- [sandboxed git leaves index.lock](reference_sandbox_git_lockfile.md) — a sandboxed `git add` can strand `.git/index.lock` → the NEXT commit's pre-commit hook dies on "File exists"/"Another git process"; retry unsandboxed before suspecting the hook
- [bats bare `!` never fails](reference_bats_bang_no_fail.md) — a top-level `! cmd` in a @test asserts nothing (shellcheck SC2314); use `run ! cmd`
- [git refuses ext:: by default](reference_git_ext_transport.md) — `protocol.ext.allow=never` is the default, so a submodule clone of `ext::sh -c …` dies with "transport 'ext' not allowed"; an ext:: injection finding needs a loosened config — probe the policy, and separate shell injection from transport selection
