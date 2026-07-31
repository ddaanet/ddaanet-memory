---
name: no-placeholder-probing
description: "Never discover a side-effecting tool's schema by feeding it a placeholder value; the call executes"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: de3d2151-a51d-4e13-8bd4-cee0e4c48775
  modified: 2026-07-29T16:04:07.000Z
---

A tool call has no dry-run mode. An argument that fails validation costs
nothing, but one that *passes* is applied — so discovering what a field
accepts by sending a throwaway value applies the throwaway value.

**Why:** the intent behind a value is invisible to the tool. A placeholder
chosen to be obviously meaningless (`"no"`, `"test"`, `"x"`) is an ordinary
value to whatever receives it, and for anything outward-facing — a rename, a
push, a message, a post — the probe *is* the action.

**How to apply:** when a required field's meaning is unclear, get it from the
tool description or `--help`. When neither answers, send the value you
actually want, so a call that lands is a call worth landing. Never send a
value chosen for what it reveals rather than for what it does.
[[tests-must-go-red]] is the mirror image: there an assertion is
trusted only once observed failing; here a call is made only once its effect
is understood.
