---
name: feedback_correction_scope
description: "a correction applies to what it names, not the class it belongs to; re-read its words before acting, and say out loud what you are NOT changing so an over-reach is caught before it lands"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 40ca6ee7-236b-45e8-9428-2f7bbdd25844
  modified: 2026-07-26T18:16:56.536Z
---

When told that a specific instance was wrong, change that instance. Do not
demolish the general claim it belonged to.

The incident: told that some odd dotfiles in this repo's working tree were
real local files, cleaned with `git clean -f`, and to remove references to
*those files*. That got read as "the sandbox-surfaces-phantom-dotfiles claim
is false" — and a verified portable fact was stripped out of a shared memory
tier and pushed, where any other repo fetching the tier would have lost it.
The correction had named a local coincidence; what it hit was the mechanism.

**Why:** a correction arrives with an implied "you were wrong", and the
cheapest-feeling repair is to clear the whole neighbourhood of the thing
named. That reads as thorough and is actually destructive — the general claim
usually survives, with the corrected instance as its exception. The asymmetry
matters: leaving a stale sentence costs a later edit, while deleting a true
one costs the fact, and in a shared store costs it for every consumer.

**How to apply:** before acting, quote the correction's own scope back and
name the delta between what it says and what you are about to change. When it
names an instance, ask explicitly whether the general claim still holds. Then
state what you are *not* touching, so an over-reach is visible while it is
still cheap to stop. Distinct from [[feedback_verify_at_source]]: there the
source was never consulted; here it was, and then over-read.
