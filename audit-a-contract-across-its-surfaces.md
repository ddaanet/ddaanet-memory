---
name: audit-a-contract-across-its-surfaces
description: "reviewing against a finding list, or a contract whose rule, enforcer and consumer live in separate files: read the three surfaces against each other — per-site verification passes while every seam goes unchecked"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: f875be65-1ce8-4ddf-9c88-54f7ce70b86e
  modified: 2026-09-18T10:26:21.341Z
---

A contract that is specified in one file, enforced by an agent or hook in a
second and consumed in a third is reviewed by reading the three against each
other. Verifying each site on its own terms confirms every fix and checks no
seam.

Name the three surfaces before reading anything, then ask of each pair:

- Does the enforcer check what the rule actually says?
- Can the enforcer repair what it detects without destroying what it just
  verified?
- Does the consumer depend on a clause nothing checks at the consumer's end?

The productive question is *what does this surface assume the other one did*.
"Is this surface defensible on its own" is what a per-site review already
answers, and it is where the misses hide.

**Why:** a deliverable review of a provenance contract — a rule file, two
reviewing agents enforcing it, one skill consuming it. Nine findings, two
delegated reviews and a first pass down the finding list all confirmed every
fix at its own site. Reading the three surfaces against one another turned up
four majors none of them could reach: the enforcers' repair directive
overwrote the one field it had been told to trust, the same directive barred
the enforcer from the one repair that was not the thing being banned, and the
consumer never checked the rule at all — so the silent failure the contract
existed to close was open on every side but the author's own discipline.

The same shape one level up from a stale-reference sweep scoped to the diff
under review, which misses drift an earlier change left in the same files
(`craft:removing-cleanly` carries that rule): there the under-covering scope is
the diff, here it is the finding list. Both fail the
same way, by reporting clean over exactly the ground they did not look at.
See [[examine-evidence-drift]] on establishing which of two co-maintained
surfaces is authoritative before deciding one is stale.
