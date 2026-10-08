---
name: cc-mods-api
description: "Claude Code mods (v2.1.287+): JS/TS module via `hooks.json` `modules`; `$.command.run` runs built-in slash commands but rejects inside a hook the turn awaits; `$.fs` has no rename/delete; every `$` call is stubbed in `claude plugin test`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4ed9b2a0-c9e5-477c-b4b0-766a8b813b72
  modified: 2026-10-08T17:55:00.964Z
---

A **mod** is a plugin whose `hooks/hooks.json` carries
`"modules": ["./x.ts"]` (beside ordinary `hooks`), exporting
`register(on, options)`. ES module, `.js`/`.ts`, no build step. Needs CC
v2.1.287+; off under `--safe-mode` / `disableAllHooks`. Docs:
`https://code.claude.com/docs/en/plugins/mods/{overview,api,reference,events,test}.md`;
full API in `anthropics/claude-code` `mods/types/claude-code.d.ts` (~500 KB —
WebFetch in 100k-char offsets), or the copy CC writes for your build into the
mod's `.claude-plugin/types/`.

Facts that change designs (d.ts read 2026-10-08, CC 2.1.294 installed):

- **`$.command.run({command, args})`** "runs a slash command as if the person
  typed `/command args`" — built-ins included (`clear`, `compact <focus>`,
  `rename <title>`). "Rejects an unknown name, and inside a hook the turn is
  waiting on" — so from `classic.Stop` return `next(e)` and defer with
  `$.clock.after(0, …)`.
- **`$.session.compact({instructions})`** "rejects while a turn runs".
- **`$.prompt.submit({text, asUser})`** runs when the session is idle;
  `asUser: true` drops the "sent by mod X" preamble.
- **`$.model.fork({prompt})`** — one request over the current conversation,
  same model/system prompt, mostly cache reads; resolves to reply text.
- **`session.start` never fires after `/clear`**; `session.end` does
  (`reason: 'clear'`). Settings hooks are `classic.<Event>` with stdin JSON as `e`.
- **`$.fs`** = `read, write, list, exists, stat, ancestors` — no rename, no
  delete, and `write` is not atomic. Atomic state files need `$.process.run`.
- **`$.process.run(argv)`** — no shell, 30s default timeout, resolves
  `{exitCode, stdout, stderr}`.
- **Testing:** `claude plugin test` runs `*.test.ts` importing
  `claude-code/testing`; *every* mods API call the mod makes needs a stub
  (`on('process.run', …)` returning `{value}`), so the kit alone is not an
  integration test. `claude plugin validate --strict` for static checks.
- **`/restart`** is a built-in (confirmed by my human partner on 2.1.294) though
  absent from the docs' command table at v2.1.290.
