# Shared ddaanet conventions

Conventions binding on all work in every repository that mounts the `ddaanet`
memory tier. Imported into each repo's `CLAUDE.md` with
`@memory/ddaanet/shared-claude.md`, so it loads whole, every session, in every
one of them.

Deliberately not named `CLAUDE.md`: a file by that name here would be injected
whenever an agent touches this directory, which is the memory store and not a
place conventions apply.

**This file is the always-in-context tier.** It is the home for a rule that
must act without being looked up — the scope between a repo's own `CLAUDE.md`
and `~/.claude/CLAUDE.md`. It is not the memory index: a fact an agent should
*find* when it meets a symptom belongs in a memory file with a routing line,
not here. Every line added here is paid for by every session in every ddaanet
repo, and unlike the memory index nothing warns when it grows. Keep it to
rules that change what the agent does by default.

## Standing defaults

- **Never call `AskUserQuestion`.** Ask inline in numbered plain prose, each
  question carrying a stated default so the thread proceeds without a
  round-trip.
- **Don't ask for obvious fixes.** A fix that is clearly correct, inside the
  agreed task, and reversible gets applied and reported, not proposed. Reserve
  questions for genuine forks — venue, strategy, anything outward-facing.
- **My human partner owns branching.** Never create or switch branches — or
  worktrees — unprompted, even when a workflow skill says to start feature
  work on a fresh one. Work on whatever is checked out. Merging onto a base they named is
  executing their instruction, not originating a switch.
- **Never `--no-verify`.** Let every hook run and surface a block instead of
  bypassing it; the parent repo's pre-commit hook is the gitlore memory gate.
  A push failing with `.git/gitlore-pre-push: exec: …/scripts/git-hooks/pre-push:
  not found` is a stale hook path, not a reason to bypass: run
  `(cd /path/to/repo && claude -p ping)` to let the SessionStart hook rewrite
  the shim and repoint `git config gitlore.hooksDir`.
- **Run the repo's precommit gate before committing**, unprompted — it is a
  required step, not a suggestion to float. In micro that is `just precommit`.
- **Commit `memory/` with the code**, never `.gitignore` it.
- **Refer to the user as "my human partner"**, never by name, in anything I
  author as myself: memory files, gate summaries, handoffs, internal docs.
  Content written *in their name* for an external audience follows the
  project's own voice instead, and git authorship carries their public
  identity by design.
- **Drafts go to a file**, not inline in the reply — GitHub comments, emails,
  documents, copy. Edit the file across iterations and describe only what
  changed.
- **GitHub bodies are not hard-wrapped**: one physical line per paragraph,
  because every newline renders as `<br>`. Blank lines between paragraphs,
  list items and headings are still correct — the rule is about
  mid-paragraph newlines. No hand-written files-changed list.
- **Commands printed for a human to run must be verbatim-runnable.**
  Substitute every real path and sha; where several commands share a long
  root, bind it to a variable on the block's first line rather than leaving a
  placeholder. When the block exists to preserve something, state the check
  that proves it worked — a mid-block failure is silent when the commands
  either side succeed.
- **Directives emitted for a subagent must be self-contained**: absolute
  paths, an explicit `cd "<dir>" &&` where CWD matters, every `$VAR` resolved
  at emit time. A subagent shell inherits neither the environment nor the CWD.
  Bats tests with `set -e` and pre-set env vars routinely mask this; verify by
  running the directive verbatim under `env -i` from an unrelated CWD.
- **A dispatch whose output matters names an output file.** The subagent
  writes its report there and replies with one line pointing at it; read the
  file, not the message. An idle reply is not evidence the agent died — check
  the artifact, `git log`, and the tree before re-dispatching. Never fall back
  to reading the subagent's transcript: it is the full JSONL conversation and
  overflows the context window.
- **No volatile git state in memory files** — no commit ids, branch tips, or
  "uncommitted as of" notes. Identify things by durable names: branches,
  paths, issue numbers. Upstream-main shas are stable but still discouraged.
  A session that needs exact volatile state puts it in a handoff, not memory.
- **"Turn" means a user turn** — one user message and everything that follows
  it until the next user message — not an API round-trip and not a tool call;
  "call" covers both API and tool calls.

## A brief's recommendation is input, not a decision

A brief that enumerates options and recommends one has not decided anything.
Surface the choice and get David's call before writing code, even when the
brief's own stated constraints appear to settle it.

Briefs are usually written by an agent in another repo, so both the
recommendation and the constraints justifying it are that agent's assumptions
rather than verified requirements. A constraint can be flatly untrue —
"backward compatibility matters more than elegance" reads as decisive and is
the opposite of how the plugin-dev toolkit is designed. Implementing the
recommended option on that basis costs a full cycle and lands the wrong design.

Treat a brief's preference order and constraint list as input. Verify the
recommendation cheaply — empirical probes are worth running whichever option
wins — then ask which to implement. Keep doing the non-contingent work while
waiting rather than blocking outright.
