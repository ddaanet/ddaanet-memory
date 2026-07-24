---
name: ""
description: Cross-project facts shared by all ddaanet repositories
metadata: 
  node_type: memory
  originSessionId: 559f871e-4271-4a30-a460-4ed106886f7e
  modified: 2026-07-24T17:56:48.554Z
---

# ddaanet tier index

Portable facts that hold across every ddaanet project — tooling behaviour,
Claude Code mechanics, shell and git gotchas. Project-specific facts stay in
the project's own memory store.

- [sandbox effects](reference_sandbox_effects.md) — index.lock/"Another git process", phantom dotfiles, sibling-worktree read-only, --resume "No conversation found…", $TMPDIR unset, zellij "no active session"; escapes: dangerouslyDisableSandbox, /add-dir, `!`
- [bats + shellcheck gotchas](reference_bats_shellcheck_gotchas.md) — SC2314 bare `! cmd` asserts nothing (use `run !`); `run missing_fn` exits 127 so negatives pass vacuously; @test bodies run under errexit; shellcheck lints .bats; `# shellcheck` comment = directive
- [git refuses ext:: by default](reference_git_ext_transport.md) — `fatal: transport 'ext' not allowed`; `protocol.ext.allow` defaults `never`, so an `ext::sh -c` injection finding isn't exploitable on default git — probe the policy first
- [protocol.file.allow blocks submodule clones](reference_git_protocol_file_allow.md) — `fatal: transport 'file' not allowed`; the child *clone* reads `protocol.file.allow`, so repo config can't fix it — use `git -c` or `GIT_CONFIG_COUNT/KEY_0/VALUE_0`
- [mutation-check negative assertions](feedback_mutation_check_negatives.md) — an exemption/"stays silent" test passes vacuously against code that doesn't exist; break the guard it protects and watch that test go red before counting it
- [shell-gotchas before reviewing shell](feedback_shell_gotchas_on_review.md) — load `/shell-scripting:shell-gotchas` before reviewing `.sh`/`.bats`/hook diffs, not only before writing them; shellcheck cannot see the BSD/`set -e`/hook-env class, and the skill's "audit the whole touched script" changes the unit of review
- [design-doc writing](feedback_design_doc_writing.md) — dated design entries are write-time records (keep rationale); memory/docs/comments state present-tense truth; six-section doc: FR, NFR, architecture, decisions, rejected alternatives, changelog
- [skill vs command](feedback_skill_vs_command.md) — self-triggering skill when condition is mechanical/detectable; command only for explicit user-initiated actions
- [verify claims at their source](feedback_verify_at_source.md) — verify any claim against its own source before acting: subagent findings, delegated citations, hook-cited limits, handoff/memory status claims, user corrections, spec symbols
- [examine evidence for drift direction](feedback_examine_evidence_drift.md) — when two co-maintained artifacts diverge, get the drift direction from git history, don't assume a source of truth
- [loose generation + post-hoc fix](feedback_loose_generation.md) — let generator scaffold freely, constrain at artifact boundary
- [user: technical depth on LLM mechanism](user_technical_depth.md) — expects rigorous attention/cost/training claims, not hand-waving
- [CC interleaved thinking](reference_cc_interleaved_thinking.md) — thinking blocks are the hidden scaffolding channel (TUI renders text/tool_use only); older models need `alwaysThinkingEnabled: true` in settings.json
- [decide/plan as late as possible, but not later](feedback_plan_late.md) — defer until evidence arrives, act before inaction decides by default
- [CC project-dir encoding](reference_cc_project_dir_encoding.md) — `~/.claude/projects/<name>/` = abs path with `[^A-Za-z0-9]`→`-`; to find a transcript search `<session-id>.jsonl`, don't apply the rule; edify's impl incomplete
- [outside-in TDD when architecture is fixed](feedback_outside_in_tdd.md) — red e2e first, drive units by failure; black-box tests survive refactors
- [dogfood early](feedback_dogfood_early.md) — run new code on the actual production target the day it ships; fixtures miss real-world bugs
- [default to automation](feedback_automate_default.md) — automated tests by default; when a manual check is justified
- [self-contained directives](feedback_self_contained_directives.md) — emit absolute paths + explicit cd; never rely on sub-agent's env or CWD
- [CC agent discovery](reference_cc_agent_discovery.md) — cwd `agents/` not auto-loaded; plugin must be marketplace-installed OR `--plugin-dir`-loaded for sub-agent dispatch; address as `<plugin>:<agent>`; defs cached per-session
- [large docs review](feedback_large_docs_review.md) — for big plan/spec docs, flag high-risk sections explicitly rather than asking for full eyeball pass
- [plan length matches work](feedback_plan_length_matches_work.md) — plan doc length tracks the executable work, not the topic
- [git hook env leak](reference_git_hook_env_leak.md) — hooks inherit repo-local `GIT_*`; clear the full set via `unset $(git rev-parse --local-env-vars)`, except `GIT_INDEX_FILE`, which a staging hook must restore for its `git add`
- [submodule escape to parent](reference_submodule_escape_to_parent.md) — git -C/cd into an unchecked-out submodule operates on the parent; guard with `[ -e path/.git ]`; linked-worktree = git-dir≠common-dir
- [CC sub-agent approval](reference_cc_subagent_approval.md) — `No agent named 'X' is currently addressable`: a one-shot sub-agent has no parent to SendMessage; gate approvals by returning, then parent `SendMessage(agentId, …)`
- [no hand-run tests](feedback_no_handrun_tests.md) — encode behavior in the bats suite; manual runs are for dogfooding the real product
- [harden human gates, not controlled channels](feedback_harden_human_gates.md) — apply word-boundary/escape recognition where a human types free-text, not on channels whose producer you control
- [CC command namespacing](reference_cc_command_namespacing.md) — `/<plugin>:<path-under-commands>`; keep commands flat or you get a double prefix
- [don't edit another repo in place](feedback_no_in_place_other_repos.md) — other repos stay read-only: investigate and propose a patch; the consent boundary is the working tree
- [stale plugin code](reference_stale_plugin_code.md) — running plugin code ≠ what you edited: `/plugin update` no-ops on unchanged version; skills+hooks need `/reload-plugins`; hooks.json events frozen at session start, settings.local.json not — restart
- [plugin recurse-clone submodule](reference_plugin_recurse_clone.md) — `/plugin install` recurse-clones; a committed submodule needs an absolute, public url or install aborts
- [hourglass display + stopped-state caching](reference_hourglass_display_caching.md) — displayed time = last API call + 1h; stopped states cached ~60 min
- [`!` shell shares agent cwd](reference_bang_shell_shared_cwd.md) — user's `!` command runs where the agent's last `cd` left the shell; make `!` commands cwd-independent
- [session title via custom-title transcript entry](reference_session_title_customtitle.md) — no hook-output field; `/rename` writes a `custom-title` jsonl line; a hook can append one to `transcript_path`
- [PostToolUse additionalContext in --print mode](feedback_posttooluse_print_mode.md) — under --print, PostToolUse hooks fire and wrapped hookSpecificOutput.additionalContext injects (tested 2.1.212); SessionStart/PreToolUse/PostToolBatch fire under --print too (2026-07-22), so a --print harness can drive the whole hook set; --print --resume does scripted multi-turn; SDK `query()` is stateless (spawns per call, `resume` replays) so BOTH harnesses re-prime every turn — measured 2026-07-17, SDK $0.379/14.4s vs --print --resume $0.425/24.3s per two-turn trial; pick on dependency footprint, not context reuse
- [probing CC behavior via tmux PTY](reference_cc_behavior_probing_tmux_pty.md) — drive interactive claude in a bash tmux pane + inspect transcript JSONL; --disallowedTools tool-gating; --settings overrides gitlore hook
- [verify the session root](feedback_verify_session_root.md) — after a compaction, check PWD / CLAUDE_PROJECT_DIR / gitStatus before acting; the 2026-07-20 wrong-root incident
- [hooks cannot inject tool calls](reference_hook_cannot_inject_tool_calls.md) — no `injectToolCall`/`forceRead` field exists (grepped 2.1.217); have the hook read the files and emit the bytes as `additionalContext` instead
- [hook output channels](reference_hook_output_channels.md) — systemMessage = user-visible; additionalContext = model-only, honoured at PreToolUse too (arrives even when the call fails); stderr only on non-zero exit; hook ERROR = systemMessage + exit 0; only exit 2 blocks, PreToolUse only
- [CC worktree memory freeze](reference_cc_worktree_memory_freeze.md) — in-process EnterWorktree moves cwd but freezes launch env (PATH/autoMemoryDirectory/CLAUDE_PROJECT_DIR); auto-memory strands on launch repo; cwd≠CLAUDE_PROJECT_DIR is the D15 drift signal; Enter/Exit confirmed to fire PostToolUse (targeted matcher)
- [prohibition vs procedure in always-on context](feedback_prohibition_vs_procedure.md) — front-load prohibitions, defer procedures to their trigger; a resident recipe turns into ceremony
- [test the invocation path, not just the code](feedback_test_the_invocation_path.md) — assert discovery, `[ -x ]`, and that the suite is listed in `make test`; the 100644 hook and 5 unlisted suites
- [git replay hooks](reference_git_replay_hooks.md) — `rebase --continue` does NOT fire `pre-commit` (2.47.3) but cherry-pick/revert `--continue` and a hand commit/`--amend` at the stop DO; a non-zero exit aborts the REBASE, not a commit; detect via `--git-path rebase-merge|rebase-apply|CHERRY_PICK_HEAD|REVERT_HEAD`; `MERGE_HEAD` is not a replay
- [no AskUserQuestion](feedback_no_askuserquestion.md) — David opted out of the tool; ask in numbered plain prose with a stated default
- [no stderr suppression](feedback_no_stderr_suppression.md) — drop `2>/dev/null`; prefer a guard, then capture-and-match, then a redirect justified inline; never guess in place of git's message
- [whitespace safety is essential](feedback_whitespace_safety.md) — a whitespace-unsafe parse is never a nit; test against a spaced input before ranking severity; default to NUL-delimited forms
- [git stderr + parsing mechanics](reference_git_stderr_and_parsing.md) — which git queries are silent on normal-absence (drop the redirect); `(fetch first)`/`(non-fast-forward)` = divergence vs `(pre-receive hook declined)` = policy; `fetch -q` prints NOTHING on rejection while `push -q` still reports — never classify under `-q`; `--git-common-dir` is relative in a main worktree → CDPATH trap; `-z --get-regexp` for whitespace-safe values
- [Justfile shebang indentation](feedback_justfile_shebang_indent.md) — shebang recipe lines must share a common indent; column-0 body lines break parsing, don't blame the embedded language
- [Harness over agent](feedback_harness_over_agent.md) — first rule of skill design: deterministic mechanical work belongs in hooks, not in the skill body
- [claude-plugin-dev toolkit](project_claude_plugin_dev.md) — shared release infra for ddaa's plugins; v0.2.0 added VERSION file SOT, self-release recipe, marketplace bump in consumer release
- [tmux CC mode + popups](feedback_tmux_cc_popup.md) — `tmux display-popup` invisible in iTerm2 `-CC` mode; use `tmux new-window` for interactive TUI overlays
- [revdiff plugin](reference_revdiff_plugin.md) — `revdiff@revdiff` installed for in-session diff review via `/revdiff:revdiff`; empty annotations = approved
- [JSONL sidechain segregation](project_jsonl_sidechain_segregation.md) — main session JSONLs don't interleave sidechains (verified 2026-05-17); sub-agents live in `archived/agent-*.jsonl`
- [Token counting (2026)](reference_token_counting.md) — Anthropic has no exact offline tokenizer for Claude 3+; cc-token is maturest CLI with cache; for small artifacts prefer bytes over API tokens
- [Test automation 80/20](feedback_test_automation_8020.md) — if you've spent 80% writing tests, the 20% to automate is a small markup; default to assertion-based shell tests, not agent-driven
- [TDD red/green separation](feedback_tdd_red_green_separation.md) — in subagent plan execution, never bundle a red-task into a green-task subagent; the red-phase review is what proves the test exercises the change
- [JSONL slash-command shape](reference_jsonl_slash_command_shape.md) — slash commands are a `<command-name>/foo:bar</command-name>` wrapper string in a user entry (not literal text or a Skill tool_use); agent-invoked skills are Skill tool_use blocks; skill *bodies* inject as `isMeta:true` user entries on both paths — `isMeta` is the structural flag for all harness-injected content
- [bg session no /dev/tty](feedback_bg_session_no_tty.md) — background sessions can't launch interactive TUIs (revdiff: no /dev/tty); have the user run them and read the history/output instead of retrying the launcher
- [CLAUDE_PROJECT_DIR over cwd](feedback_claude_project_dir.md) — hook scripts must use CLAUDE_PROJECT_DIR for project root, not payload cwd; cwd drifts with shell cd and /add-dir
- [gitlore memory in git](feedback_gitlore_memory.md) — memory/ is tracked by gitlore; commit it with code changes, never gitignore it
- [Review model selection](feedback_review_model_selection.md) — use Sonnet (min) for reviewer subagents; Haiku misses subtle issues regardless of who implemented
- [No compat aliases](feedback_no_compat_aliases.md) — when removing a constant, patch all consumers directly; compat aliases keep tech debt alive
- [uv: sync only, direnv venv](feedback_uv_direnv_venv.md) — no `uv run` under Claude (sandbox/cache pain); `uv sync` only + direnv exports VIRTUAL_ENV & PATH prefix; recipes call bare `pytest`
- [Hook input schema (2.1.168)](reference_hook_input_schema.md) — base stdin = session_id/transcript_path/cwd/permission_mode?/agent_id?; every event incl SessionStart carries cwd; no worktree-root field; CwdChanged is observational (watchPaths+systemMessage only, no block/additionalContext)
- [Subagents over-applied](feedback_subagents_over_applied.md) — one-subagent-per-task is mis-granular; the axis is batching (one agent for several small/mechanical/one-loop tasks to spare main-session context), not agents-vs-inline; a shared mutable file only rules out *parallel* agents (mechanical does not)
- [Reference doc scope](feedback_reference_doc_scope.md) — a lasting guideline must stay general/reusable (neutral examples, version- not repo-grounded); repo-specific application goes in a separate point-in-time audit; confirm general-vs-specific scope before delegating research
- [Skill description: purpose first](feedback_skill_description_purpose_first.md) — skill `description:` frontmatter is TUI-visible; lead with what the skill does, then triggers/"Use when", then exclusions; never a bare trigger list; drop stale implementation-detail tails
- [Skill bundled-script invocation](reference_skill_bundled_scripts.md) — CLAUDE_PLUGIN_ROOT is UNSET in agent Bash (hooks only); every *enabled* plugin's bin/ is on PATH (cross-plugin bare-name invocation works; disabled = absent); skill body invokes bundled scripts by bare-name bin/ shim, no hook needed
- [Precommit before commit](feedback_precommit_before_commit.md) — run `just precommit` unprompted before committing; it's required, never an optional suggestion
- [Session JSONL schema reference](reference_session_jsonl_schema.md) — no official schema (docs say internal/unstable); claude-dev.tools doc + simonw/claude-code-transcripts are the maintained references; structural flags over content heuristics
- [Remove features cleanly](feedback_remove_cleanly_no_vestigial.md) — cut all machinery in one pass; no absence-tests, no vestigial fixtures/scaffolding, inline a collapsed script; if a design proved harmful the cleanup is total
- [CC TUI tmux driving](reference_cc_tui_tmux_driving.md) — mid-turn input has FOUR classes (TUI-local intercepted / harness-action queued / plugin-command own-turn / **prose injected into the running turn**); Stop + SessionStart(compact) are the authoritative gates; Stop doesn't fire on Esc (verified v2.1.215)
- [tmux test isolation](reference_tmux_test_isolation.md) — TMUX=fake does NOT isolate a test from the real tmux server (it falls back to the default socket); stub the binary on PATH instead
- [Observability isn't gated on the happy path](feedback_observability_not_gated_on_happy_path.md) — never skip a failure-reporting mechanism because the run you watched went fine; enumerate failure paths and ask who learns about each
- [Todo tool is flag-gated](reference_todo_tool_flag_gated.md) — TodoWrite superseded (not removed) by TaskCreate/Get/List/Update, killed server-side by `tengu_vellum_ash`; the affordance can vanish between sessions with no changelog entry — never name a todo tool in a plugin step
- [Reconstructable is two categories](feedback_reconstructable_two_categories.md) — by-harness (deterministic, free → drop it) vs by-model-inference from a paraphrase (lossy, fails silently → carry it); name the reconstructor before dropping anything
- [Imperative-form rule scope](feedback_imperative_form_scope.md) — the skill-development rule targets instruction subjects ("You should X"), not every pronoun; a bare grep for "you" over-reports (11 hits, 1 real); `your post-compaction self` is a deliberate keep
- [gitlore Bash-edit desync](reference_gitlore_bash_edit_desync.md) — compose/sync hooks only fire on Write|Edit tool_name; a Bash-applied MEMORY.md edit desyncs root vs. tier index until the next Write/Edit touch
- [memory index lines are functional](feedback_memory_index_lines_functional.md) — index lines serve agent-matching AND ambient awareness, not just a pointer; don't mechanically shorten them to hit a byte target, especially an advisory one
- [bundle memory with source](feedback_bundle_memory_with_source.md) — always commit a memory update in the same parent commit as the source change it documents, not as a separate commit; don't over-apply to unrelated pending work
- [user: technical depth on LLM mechanism](user_technical_depth.md) — expects rigorous attention/cost/training claims, not hand-waving
