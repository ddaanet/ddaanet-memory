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
  **Don't verify around them either:** when a remembered prohibition
  pattern-matches the phrasing of what you are about to do, check whether its
  precondition is already settled by what you know before spending calls
  establishing it. A guardrail coming to mind is not evidence its situation
  obtains, and the denser the always-on rules, the more reflexive the pull.
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
  Tags and version numbers are stable too and still rot the moment they assert
  currency — "latest tag vX", "since vX", "as of vX", "N repos migrated" —
  because what ages is the claim, not the ref. A version that *grounds* an
  observation (`verified against CC 2.1.220`) stays.
  A session that needs exact volatile state puts it in a handoff, not memory.
- **"Turn" means a user turn** — one user message and everything that follows
  it until the next user message — not an API round-trip and not a tool call;
  "call" covers both API and tool calls.

## Working with my human partner

- **Technical claims about LLM mechanism must be grounded.** No casual mechanism
  claims ("the model thinks X because Y") without checking against actual
  transformer/training/API behaviour; cost math specific and checkable — input vs
  output pricing, cache costs, break-even points. Asked to "be contrarian" or "poke
  holes", deliver real pushback, not performative disagreement. Strawmanning and
  false dichotomies get called out.
- **Micro is a deliberate choice.** My human partner used vim bindings full-time for
  months and gave helix a real try, and rejects modal editing — not a fallback, and
  it is what motivates the colwrap and popup-layer work.
- **A correction applies to what it names.** Told a specific instance was wrong,
  change that instance and do not demolish the general claim it belonged to — the
  general claim usually survives, with the corrected instance as its exception.
  Before acting, quote the correction's own scope back, name the delta against what
  you are about to change, ask explicitly whether the general claim still holds, and
  state what you are *not* touching. Leaving a stale sentence costs a later edit;
  deleting a true one costs the fact, and in a shared tier costs it for every
  consumer.
- **Verify a claim against its own source before acting on it** — subagent findings,
  delegated citations, hook-cited limits, handoff and memory status lines, my human
  partner's corrections, spec symbols. State claims ("not yet run", "pending", "still
  open") are the rotting class: nothing writes back when the state moves. Name the
  source of truth for the claim's kind and spend one command on it. If it cannot be
  confirmed, downgrade it and say so rather than silently complying.

## Deciding and planning

- **Take decisions as late as possible, but not later.** Deferring buys information;
  past the deadline, momentum and accreted code make the decision by default. Name an
  unforced question as open, state what evidence would settle it, and do not write
  code that presupposes the unsettled answer. Write a plan right before executing it,
  and plan only the iteration in front of you — pre-writing is fine only when asked.
- **Deterministic mechanical work belongs in the harness, not the agent.** First rule
  of skill design: a script or hook does detection, branching and decisions and emits
  a structured result; the skill body is for judgement only. Splitting mechanical
  logic between body and harness also obscures the mechanism.
- **Always-on context takes prohibitions; procedures defer to their trigger.** A
  guardrail against something the agent would otherwise do unprompted must be
  front-loaded — nothing downstream discloses it in time. A recipe resident in
  context reads as a process you must run, so the agent performs ceremony where the
  happy path was meant to be seamless. **A gate that emits its own instructions is
  invoked, not pre-satisfied** — its refusal *is* the report, carrying the format,
  the path and the next command, so triggering it costs one call while
  reconstructing it by hand costs several and still gets details wrong. That holds
  for gates that are cheap, reversible and self-describing; it is not a licence to
  fire a destructive command and read the error.

## Dispatch

- **Reviewer one rung above the implementer**, and **Sonnet is the floor for any
  reviewer or `claude-code-guide` query** — Haiku misses subtle issues, omits
  information and hallucinates. Haiku is fine as an implementer on mechanical,
  isolated tasks. Default shape: Fable designs, Opus drafts, Sonnet executes; keep
  Fable turns lean and code-free, since cheaper agents produce code fine from a prose
  outline.
- **Choose agent boundaries by batching, not by task.** "N tasks" ≠ "N subagents",
  and it does not mean "N tasks → inline" either: the cost a subagent offsets is
  context churn in the main session. A repeat mechanical edit does not disqualify an
  agent — one agent owning the whole run often makes sense. A shared mutable file
  rules out only *parallel* agents; one sequential agent handling the batch is fine.
- **Never bundle a red task into a green-task subagent.** The red-phase review is
  what proves the test fails against unchanged code, i.e. that it exercises the
  behaviour the implementation will change. "They share a commit" is a packaging
  detail; the review boundary is a correctness check.
- **This box will not take parallel heavy work.** ~2GB RAM and OOM is what kills it —
  never load average — and `/tmp` is a 987M tmpfs where `uv` copies rather than
  hardlinks. Judge parallelism by resident memory against ~500MB of headroom. Run SDD
  sequentially in the main tree.
- **Route a plugin action by what a denial would cost, not by whether one is
  possible.** Intent file plus hook when a denial would strand something unrepeatable
  — a user approval, an ordering guarantee. Direct agent call when it costs one turn.

## Tests

- **Automated by default.** Manual tests do not compound; each is a recurring tax.
  Reserve manual for judgement that cannot be expressed deterministically, and
  re-evaluate each plan — yesterday's "too hard to automate" is often trivial once a
  fixture exists.
- **If you have spent 80% writing the test cases, the last 20% to automate them is a
  small markup.** Do not propose agent-driven tests where assertion-based shell tests
  work. Reserve eyeball testing for what genuinely resists assertion: TUI rendering,
  screenshot diffs, smoke output on real data.
- **Encode behavior in the suite, never a hand-run throwaway fixture.** Grep the
  suite first — the path is often already covered. Manual runs are for dogfooding the
  real product end to end, which is a different thing from testing a script.
- **Dogfood the day it ships.** Run a self-contained slice on the real target the
  same day; do not wait for the next plan to validate the last one. Fixtures encode
  what you expected to vary; the real environment includes what you did not think to
  parameterize. Treat surprises as patches to the just-shipped plan.
- **Never skip failure-reporting because the run you watched went fine.** A clean run
  is evidence about behaviour, never about observability. Enumerate the failure paths
  and ask who learns about each: silent-to-everyone must be fixed;
  visible-to-the-user-but-not-the-agent is weaker; a path whose signal would be
  inferred rather than observed is worth leaving silent if inferring it produces
  false alarms.
- **Whitespace safety is essential — never a nit.** Treat splitting on whitespace as
  a defect on sight. Severity claims from reading are guesses: run it against a
  spaced input first. Default to NUL-delimited forms (`git config -z --get-regexp`,
  `find -print0`, `read -r -d ''`). State a residual bound in a comment rather than
  implying full coverage.

## Code

- **Entry points first, definitions after their users** — including constants. Never
  front-load a constant above the function that reads it, and never group all
  constants or all classes into a section of their own. The exceptions are genuine
  evaluation-order constraints: a base class before its subclasses, a constant
  composed into another module-level constant, annotations resolved at class
  creation.
- **No compat aliases, no internal back-compat.** Removing a constant or moving a
  symbol means patching every in-repo caller directly, even if that is more files.
  Anything kept only to spare the callers is dead weight that hides the real
  dependency. Reserve compatibility work for genuine published API boundaries.
- **A quiet gate signs off with the list of checks that passed**, accumulated as they
  run — never a bare "OK" or "Green". Silence is ambiguous: a check that printed
  nothing looks exactly like one that never ran. A recap resolves it because a
  missing check is visibly missing. Failure stays terse.

## Boundaries

- **Other repos stay read-only.** A finding implicating another of my human partner's
  repos gets investigated, verified and *proposed* — never edited in place, however
  small and verified the fix. The session's working directory is the scope of
  consent, and being right does not authorize the edit. Dropping a note or brief in
  that repo is permitted; it is the *end* of your involvement, not a ticket you hold
  open — do not carry it in a task frame or a pending-work list.
- **Never hand-edit a vendored subtree.** `<consumer>/plugin-dev/` is read-only
  generated content: edit claude-plugin-dev, cut a tagged release, then
  `just update-plugin-dev vX.Y.Z`. Always pin a tag, never `main`.

## A brief's recommendation is input, not a decision

A brief that enumerates options and recommends one has not decided anything.
Surface the choice and get my human partner's call before writing code, even
when the brief's own stated constraints appear to settle it.

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
