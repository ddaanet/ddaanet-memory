---
name: codex-model-full-ids
description: "Codex `--model` short names (astra, sol) fail with \"not supported when using Codex with a ChatGPT account\" — a misleading error; pass the full ID from ~/.codex/models_cache.json"
metadata:
  node_type: memory
  type: reference
  originSessionId: f27bcb1b-f41e-43b3-aaad-8d2014aef41a
  modified: 2026-10-29T20:03:59.233Z
---

Passing a short model name to the Codex companion
(`codex-companion.mjs adversarial-review --model astra`, or `sol`) fails at
the first turn with
`The 'astra' model is not supported when using Codex with a ChatGPT account.`
(`model_not_found`). The message blames the account, and re-logging in or
upgrading the plan changes nothing: the name is the problem. The full IDs
(`gpt-6-astra`, `gpt-6.1-sol`, …) work on the same ChatGPT login.

List the valid IDs with
`jq -r '.models[] | .slug // .id' ~/.codex/models_cache.json`; the configured
default is the `model =` line in `~/.codex/config.toml`. When a short name
matches several IDs (`gpt-6.1-sol`, `gpt-6-sol`, `gpt-5.6-sol`), pick the
newest and say so.

Observed with openai-codex plugin 1.0.6, 2026-10-29.

Related: [[codex-review-of-design-docs]].
