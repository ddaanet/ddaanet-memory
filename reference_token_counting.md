---
name: token-counting
description: "Token counting in Claude Code plugin contexts — Anthropic has no exact offline tokenizer for Claude 3+, count_tokens API is the only precise path, cc-token is the maturest CLI"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 45acd379-a2c1-4cfc-8f7b-e506ad744d38
---

State of token-counting tooling as of 2026-05-19 (surveyed during
handoff SessionStart-hook design):

**Precise tokens require the API.** Anthropic has not open-sourced an
exact offline tokenizer for Claude 3+. Offline tokenizers
(`@anthropic-ai/tokenizer`, `claude-tokenizer` Rust crate,
`Xenova/claude-tokenizer`) are BPE approximations only — accurate for
older Claude versions, rough for 3+. The `messages.count_tokens` API
endpoint is the only precise option (and Anthropic's own docs note
it still returns an *estimate*, not ground truth).

**Maturest CLI with cache:** [iota-uz/cc-token](https://github.com/iota-uz/cc-token)
— Go binary, wraps `count_tokens`, built-in JSON cache at
`~/.cc-token/cache.json` keyed on SHA-256 + mtime + model.
`--no-cache` to bypass. Closest off-the-shelf fit for "small CLI
with cache."

**ddaa's own implementation:** [[claude-plugin-dev]]-adjacent —
[edify](https://github.com/ddaanet/edify) has `tokens.py` +
`token_cache.py` (SQLAlchemy keyed on MD5+model), and a `tokens`
CLI. Heavyweight for a bash hook (Python + anthropic SDK + sqlalchemy)
but useful as reference for the API-call+cache shape.

**Practical takeaway for plugin hooks:** if the artifact is small
(1–5 KiB), reporting **bytes** answers the user's "is this material
enough to care?" question just as well as tokens, with zero deps
(no API key, no network, no cache subsystem). Default to bytes unless
the artifact is large enough that the tokens-vs-bytes ratio meaningfully
shifts the answer.
