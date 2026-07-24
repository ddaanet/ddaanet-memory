---
name: reference-git-ext-transport
description: "`fatal: transport 'ext' not allowed`; `protocol.ext.allow` defaults `never`, so an `ext::sh -c` injection finding isn't exploitable on default git — probe the policy first"
metadata: 
  node_type: memory
  type: reference
  originSessionId: d7a4fa69-e3f9-4102-b9d6-7d3976d65bcf
  modified: 2026-07-22T13:49:59.424Z
---

`git-remote-ext` runs a shell command as a transport: a url of the form
`ext::sh -c '…'` is a documented remote-code-execution vector, and a standard
finding in automated security review of any code that clones a caller-supplied
url.

**It is off by default.** `protocol.allow` defaults to `never` for
known-dangerous protocols, `ext` among them. Verified against git 2.47.3 —
`git submodule add -- "ext::sh -c 'touch X'" p` dies with:

```
fatal: transport 'ext' not allowed
```

and nothing runs. Enabling it takes an explicit `protocol.ext.allow=always`.

So an "ext:: command injection" finding is **not exploitable on a default git**.
Probe the actual policy before ranking one — and separate the two claims it
usually conflates: shell injection (does the url reach a shell? not if it is a
quoted argument after `--`) versus transport selection (which git transports can
the url choose?). Only the second is live here, and only under a loosened config.

Worth guarding anyway when the clone runs somewhere the caller could not
otherwise reach — a **hook runs outside the agent's command sandbox and has
network**, so a url handed to a hook reaches further than one the agent could
use itself. A scheme allowlist (`http`/`https`/`ssh`/`git`/`file`, scp-like
`[user@]host:path`, local paths) closes the class independently of git config;
reject anything whose text before the first colon is followed by a second colon.

See [[feedback-harden-human-gates]] for the matching question of *where* a
guard belongs.
