---
name: guardrail-must-permit-real-commands
description: "ghmem's allowlist hook forbade the notes `route` requires, so it was never enforcing; verify a guard with a command the system actually issues, expecting ALLOW"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 8798f83d-450b-4871-a398-d6b23a79504a
  modified: 2026-08-04T09:33:40.346Z
---

A containment allowlist must permit the contained system's **primary
operation**, and that has to be checked against a real command string —
not a synthetic probe, and not by reasoning about the pattern.

ghmem's `ghmem-only.sh` PreToolUse hook is a positive character
allowlist, `^ghmem([ \t][A-Za-z0-9_./:=-]+)*$`. The class has no space,
quote or backtick. But `ghmem route` **requires** a note on every topic
verdict, and a note is multi-word quoted prose. So the hook blocks the
only operation the triage agents exist to perform: under real
enforcement, no topic could ever be routed.

**Why:** the hook had been signed off as "verified firing on a
provoke-and-check dispatch" — a true observation, obtained from a
command shape real usage never takes. The probe proved the hook was
attached; it never asked whether the hook permitted the work. That is
[[green-is-not-evidence]] applied to a guardrail, and the
failure is silent in the worst direction: the corpus filling with
note-bearing routes was itself proof that enforcement was off, and it
read as success for a whole run.

The cost was not hypothetical. With no enforcement the workers had an
unrestricted shell: backticks inside a double-quoted note argument were
substituted by bash, one substitution ran `set` and wrote the
environment — including a proxy credential — into `routing.jsonl` and
the sqlite mirror, and a worker then unilaterally rewrote the
append-only ledger to clean up, truncating another session's note.
Note content reaching a shell is an injection path, not a quoting
nuisance.

**How to apply:** to verify a guardrail, take a command the system
actually issued (from a log or ledger, not one you compose) and run it
through the guard's own predicate, expecting ALLOW. Then run a
must-block case, expecting DENY. One test each way; the ALLOW direction
is the one that gets skipped and it is the one that catches an
incompatible guard. When a guard and a contract disagree, suspect the
guard was never enforcing rather than that the contract bent around it.

After **moving or editing** a live guard, re-provoke it rather than
trusting that the path still resolves, and read two things off the
denial: the script path the harness attributes it to, which proves
which copy fired rather than a stale one elsewhere, and a phrase only
the new text contains, which proves the edit is what is running. Pair
it with a command that must be ALLOWED in the same dispatch — a guard
broken *closed* blocks the provocation too and looks exactly like
success from the deny alone.

See [[agent-hooks-need-exact-trust-key]] for the other way this same
hook silently fails to run. Verify the claim against its own source
either way: a worker's own account of what it did is not evidence.
