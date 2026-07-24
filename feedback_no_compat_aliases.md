---
name: feedback_no_compat_aliases
description: "When removing a constant, patch all consumers directly — don't add a compat alias"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 59a6338a-0de4-45c7-9150-a24136aca01c
---

When removing a constant or symbol, update all files that use it directly (even if it's three files), rather than keeping the old name as a compat alias in a shared helper.

**Why:** A compat alias keeps tech debt alive in the codebase; patching three files is more work but keeps the code clean. User: "patching three files is more work, but it's keeping the tech debt low."

**How to apply:** After removing a constant from a shared file, grep for all consumers and patch them individually. Only add a compat alias if backward compatibility across a published API boundary is genuinely required.
