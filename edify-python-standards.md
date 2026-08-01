---
name: edify-python-standards
description: "uv, click, pytest, ruff-ALL, mypy strict; edify repo is the reference config; justfile shape and the 400-line check_line_limits.sh gate"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2eeaac7d-8f74-4a8c-941d-66cd37826d27
---

The reference for Python tooling standards is my human partner's edify
repo: `/Users/david/code/edify/pyproject.toml` and its `justfile`
(usage-hardened settings).

Key points: uv-managed project; click for CLIs (preferred over argparse
for all but the simplest scripts); pytest (+pytest-mock) over stdlib
unittest; `ruff select = ["ALL"]` with edify's curated ignore list;
`mypy strict = true` + `extra_checks`; docformatter with 80-col wrap;
pydocstyle via ruff's D rules (pep257 convention); justfile `green`
recipe pattern (format → ruff → docformatter -c → mypy → pytest with
input-hash sentinel). "No pip dependencies" is NOT a standing
constraint — dependency-using code held to these standards is
preferred over stdlib-only austerity.

**Justfile shape** (edify + pytest-md agree; verified 2026-07-17 by
reading both):

- Recipes call tools by **bare name** (`ruff`, `mypy`, `pytest`,
  `docformatter`) — they come from the direnv-activated venv on PATH.
  Never `./.venv/bin/foo` and never `uv run foo`. The `.envrc` does
  `PATH_add "$PWD/.venv/bin"`.
- `sync` is a **bash function in the shared `bash_prolog`**, called bare
  at the top of recipes that need fresh deps — never a `just` recipe.
  `just sync` does not exist in these projects and must not be created.
  Body: `sync () { if [ -w /tmp ]; then uv sync -q; fi; }` — the `[ -w
  /tmp ]` probe is the sandbox guard (Claude Code's uv cache is
  read-only), and it works: /tmp is genuinely unwritable in-sandbox. Do
  not reinvent this with `$SANDBOX_RUNTIME`.
- `setup` is the name for the public one-time dep/install job, if one is
  needed at all.
- Prolog also carries `safe`/`end-safe` (run every check, report all
  failures, still exit non-zero) and `show`/`visible`/`fail`.
- **`scripts/check_line_limits.sh`** — a hard 400-line `wc -l` gate over
  `src` and `tests`, wired into the gate recipe. Present in edify, home,
  pytest-md. Edify is the only sampled repo where every module is under
  400 lines, and that is causal: ungated repos drift to 600–1200. Ruff's
  complexity rules (C901/PLR0912/PLR0915, active under `select=ALL`) do
  NOT substitute — a file can be 1200 lines of small functions and pass
  them all. Only a file-line gate catches breadth.
- Edify runs two tiers: `green`/`lint` relax complexity rules for
  red/green TDD iteration; `precommit`/`release` enforce them plus the
  line cap.
