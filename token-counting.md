---
name: token-counting
description: "counting a file's tokens: `edify tokens --model opus --json FILE...` (count_tokens, no key in env needed), never a `claude -p` usage round-trip; offline BPE is approximate; report bytes for small artifacts"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 45acd379-a2c1-4cfc-8f7b-e506ad744d38
  modified: 2026-10-31T14:31:26.333Z
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

**Counting tokens from a session: `edify tokens`.** It is on PATH
(`~/.local/bin/edify`): `edify tokens --model opus --json FILE...`
returns per-file counts and a total in one call (verified 2026-10-31,
`claude-opus-5-5`), and it works with `ANTHROPIC_API_KEY` unset in the
session env. Each count carries a constant +7 tokens of request
overhead, and haiku and opus counts agree exactly (measured over 131
files). Never measure tokens by a model round-trip — `claude -p
--output-format json` usage minus an empty-prompt baseline: it gives
the same numbers but pays for a generation per file, one file per
call; my human partner called it "remarkably stupid" next to the CLI
that exists. The implementation
([edify](https://github.com/ddaanet/edify) `tokens.py` +
`token_cache.py`, SQLAlchemy keyed on MD5+model) is heavyweight for a
bash hook (Python + anthropic SDK + sqlalchemy) but useful as
reference for the API-call+cache shape.

**Practical takeaway for plugin hooks:** if the artifact is small
(1–5 KiB), reporting **bytes** answers my human partner's "is this
material enough to care?" question just as well as tokens, with zero
deps (no API key, no network, no cache subsystem). Default to bytes
unless the artifact is large enough that the tokens-vs-bytes ratio
meaningfully shifts the answer.
