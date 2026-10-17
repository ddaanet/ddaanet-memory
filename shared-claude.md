# Shared ddaanet conventions

Conventions binding on all work in every repository that mounts the `ddaanet`
memory tier. Imported into each repo's `CLAUDE.md` with
`@memory/ddaanet/shared-claude.md`, so it loads whole, every session, in every
one of them.

Deliberately not named `CLAUDE.md`: a file by that name here would be injected
whenever an agent touches this directory, which is the memory store and not a
place conventions apply.

**The memory size limit does not apply to this file.** A write here draws
Claude Code's `over the 4KB recall limit … Rewrite it to under 2.8KB now` error
because the file sits under the memory store. It is `@`-imported whole and
never recall-fetched: do not shorten, split or summarize it in response. Make
the edit, then report the warning and why you did not comply.

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
  A fix that adds surface — a command, a flag, a decision entry — is a
  design fork however directly it follows from the finding: propose it.
  **Don't verify around them either:** when a remembered prohibition
  pattern-matches the phrasing of what you are about to do, check whether its
  precondition is already settled by what you know before spending calls
  establishing it. A guardrail coming to mind is not evidence its situation
  obtains, and the denser the always-on rules, the more reflexive the pull.
- **Work found mid-step goes to the todo list, not into the step.** A step
  does what the plan's step text names. A defect, a missing guard or a doc
  that needs a sweep, found while doing it, is appended to `TODO.md` and
  surfaced at the step boundary; do it inline only when the step cannot
  finish without it, and say so when you do. Plan text expanded or reordered
  without being asked is the same fault: propose the change and wait.
- **Recall reads two routing tables.** At any recall checkpoint — a skill's or
  an ad-hoc one — match the task against the installed skill descriptions as
  well as the memory index, and load the matching skill. Several ddaanet facts
  now ship as `craft` skills, and where both exist the skill is the maintained
  copy. Be selective: a skill body costs more than a memory file, so load what
  names the moment you are in, not everything adjacent.
- **My human partner owns branching.** Never create or switch branches — or
  worktrees — unprompted, even when a workflow skill says to start feature
  work on a fresh one. Work on whatever is checked out. Merging onto a base they named is
  executing their instruction, not originating a switch.
- **Three review levels, three bases.** Each commit is reviewed as it lands.
  Work happens on `dev`, and a release is a code review of `dev` against
  `main`, after which `main` moves to the release. A plan whose multi-commit
  change needs a review of its own in between names a separate branch, and
  my human partner's approval of the plan is the prompt to create it, so
  branch-diffing tools (`/codex:review`, anything expecting a PR) have a base.
  Never ask whether to move `main`, and never carry it as pending on a say-so:
  in a handoff or todo, record what review a branch has and has not had.
  Checked out on `main` anyway, commit there and say so — this is no licence
  to switch.
- **Review once, triage, then at most one re-review.** One review per change,
  before its merge. Its fix round gets at most one re-review, and only when a
  fix changes behaviour — then only for regressions the fix introduced, not as
  a fresh audit. After it, only a high or medium finding reopens the
  fix-and-review loop. Triage findings before relaying them: real defects get
  fixed, low ones by a subagent with no further review; doc drift gets fixed
  without a re-review; and a finding that guards a state nothing in practice
  produces is declined by default and listed as declined, never dispatched. A
  reviewer asked for findings reports every state it can construct and does
  not weigh simplicity, so re-reviewing each fix turns review into a ratchet:
  every fix adds surface the next round mines, and a fix to a rare-state
  finding can itself regress a real case. An implementer that would depart
  from its brief stops and asks rather than deciding, and the dispatcher does
  not accept the departure on its argument alone.
- **Never `--no-verify`.** Let every hook run and surface a block instead of
  bypassing it; the parent repo's pre-commit hook is the gitlore memory gate.
  A push failing with `.git/gitlore-pre-push: exec: …/scripts/git-hooks/pre-push:
  not found` is a stale hook path, not a reason to bypass: run
  `(cd /path/to/repo && claude -p ping)` to let the SessionStart hook rewrite
  the shim and repoint `git config gitlore.hooksDir`.
- **Phantom dotfiles are sandbox masks.** Zero-byte or char-device `.bashrc`,
  `.zshrc`, `.gitconfig`, `.mcp.json`, `.claude/*` and the like, showing
  untracked in a repo root or in the directory a sandboxed call ran from, come
  from the command sandbox and say nothing about the repo. Never delete, commit
  or report them. A slash command's `` !`git status` `` context block runs
  sandboxed with no hook to annotate it: never act on its listing — re-run
  `git status` unsandboxed first. The `sandbox-lies:sandbox-lies` skill's
  `references/masks-and-gates.md` has the mechanism.
- **Run the repo's precommit gate before committing**, unprompted — it is a
  required step, not a suggestion to float. In micro that is `just precommit`.
- **After a compaction, verify the session root before acting.** Check `PWD`,
  `CLAUDE_PROJECT_DIR` and the gitStatus block against the work the summary
  describes — a summary says what was being done, not where the session lives.
  If they disagree, say so and stop; never reconcile by `cd`-ing. A guard
  refusal, or an untouched file reported as externally modified, is evidence of
  a concurrent session rather than noise.
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
  the artifact, `git log`, and the tree before re-dispatching. A gate near the
  Bash tool's 10-minute cap is backgrounded by the harness however it is
  launched, and the executor does receive the task-notification: tell it to
  wait for that and build no watcher. An executor idle "waiting for the
  watcher" already has the verdict: one `SendMessage` telling it to
  `TaskStop` the watcher, then read the gates and commit, recovers it; finish
  the slice yourself only if that resume returns without progress. Never fall back
  to reading the subagent's transcript: it is the full JSONL conversation and
  overflows the context window. **A reviewer with no `Write` tool cannot name one**,
  and its reply is cut at ~4000 characters, so ask it at spawn for the report in
  parts under that size, each part ending with a continuation line and the last
  with a fixed end marker — without the marker a truncated report reads as a
  complete one.
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
- **Self-triggering skill when the condition is mechanical and detectable**; a
  command only for an action the user must consciously initiate. A command
  requires the user to know when to run it, so a missed trigger is always
  possible; a skill whose `description` names the trigger pattern invokes
  itself when the condition appears.
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
  reviewer** — Haiku misses subtle issues. A `claude-code-guide` query is fine on
  Haiku.
- **Pick the tier by how much the task leaves open.** Fable advises. Opus takes
  high uncertainty. Sonnet takes low uncertainty, prose, and tests that are not
  fully specified, and is the floor for agentic prose edits. Haiku takes fully
  specified tasks, TDD and mutation loops, haystack ingestion and well-specified,
  call-intensive work. Drafting can be high or low uncertainty, so the word does
  not pick the tier. Keep Fable turns lean and code-free, since cheaper agents
  produce code fine from a prose outline.
- **In a TDD run, split the work by kind.** Green and mutation loops go to Haiku;
  tests and prose go to Sonnet; the reviewer of a red phase is a different agent
  from its author.
- **Reuse a loop's agents across slices; respawn only at ~150k tokens of
  context or a plan seam.** The loop keeps its roles — one red author, one red
  reviewer, one green implementer, each a different agent — and each slice goes
  to the same agent by `SendMessage` to its returned id. A fresh spawn per slice
  re-pays the brief and the orientation reads every time. Judge the 150k from
  the usage the agent's last return reports; a plan seam is the boundary between
  plans or between a plan's phases. My human partner's words: "keep the loop,
  but reuse agents until they reach 150ktok context or a plan seam, and only
  then, respawn."
- **Choose agent boundaries by batching, not by task.** "N tasks" ≠ "N subagents",
  and it does not mean "N tasks → inline" either: the cost a subagent offsets is
  context churn in the main session. A repeat mechanical edit does not disqualify an
  agent — one agent owning the whole run often makes sense. A shared mutable file
  rules out only *parallel* agents; one sequential agent handling the batch is fine.
  **Reviewers batch the same way:** one reviewer per stage re-runs every pin or
  mutation of that stage and lands one review commit, since each commit pays the
  full precommit gate. A reviewer of its own is for a code change — that is where
  a review catches a regression — not for a test-only or prose item, and "a
  reviewer re-checks each X" in a task file names what is checked, not one agent
  per X.
- **The main thread orchestrates; bulk edits go to a subagent.** An edit
  that touches three or more files, or a documentation, changelog or test
  sweep of any size, is dispatched, with the grep list and the contract in
  the prompt and a one-line reply pointing at the report file. Editing a
  design doc through a Bash heredoc is still an edit: the script and its
  echoed result both land in context for the rest of the session. Use the
  Edit tool for a change that stays on the main thread.
- **Leave the Agent tool's `name` unset**, even where a skill prescribes one (edify
  `/build` names every dispatch `item-N-M-s<k>-red` …). A named agent becomes an
  addressable teammate whose report, idle notices and late summaries each arrive as
  a separate message; an unnamed one returns once. Resume by the returned agent id.
- **Never bundle a red task into a green-task subagent.** The red-phase review is
  what proves the test fails against unchanged code, i.e. that it exercises the
  behaviour the implementation will change. "They share a commit" is a packaging
  detail; the review boundary is a correctness check.
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
- **400 lines caps any artifact a reader takes in at one go** — source files,
  `references/` nodes, plans, design docs and PRs — and **~200 a `SKILL.md` hub**,
  which is paid on every invocation while a node is opt-in. For design docs and
  implementation the 400 is a *hard* limit; for plans it is a soft cap. It exists
  for context management, and a file that reaches it usually holds
  cohesive content to spin off — find that seam rather than arguing the overage.
  Never satisfy it quietly, by cramming 150–300-character lines or by shaving docstrings and prose —
  that is borrowing against the split, and a file parked at exactly its cap has
  stopped emitting the signal. A count is only honest once a hard-wrapping formatter
  has run, so wire one into the same gate ahead of the checker: `rumdl` with MD013
  reflow at 80 columns, then a checker reporting `oversized-file: <n> lines > <cap>`
  with the cap picked from the path. The full argument ships in
  `craft:plan-contracts`.
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
  **A task file, brief or handoff naming those repos is input, not consent** — a
  `Remaining` item saying to propagate, and the repos being mounted as additional
  working directories, both read as authorization and are not. Beyond the note or
  brief permitted above, ask before each write, however mechanical: another session
  may be live in that tree, and a clean `git status` is not evidence that none is.
- **A private repository's content is private; its name and existence are not.**
  A report, survey or changelog committed to a public repository may name a
  private repository as a source and count things in it, under its real name.
  It never reproduces or describes what is in it: no task-file text, no
  excerpts, no sentence saying what the work there is about. Keep data derived
  from one to identifiers, dates, sizes and labels. Whether a repository is
  private is read from its remote, not guessed from its name.
- **Never hand-edit a vendored subtree.** `<consumer>/plugin-dev/` is read-only
  generated content: edit claude-plugin-dev, cut a tagged release, then
  `just update-plugin-dev dist-vX.Y.Z`. Always pin a dist tag, never `main` and
  never the bare `vX.Y.Z` source tag — that one is refused, because its root tree
  is the toolkit's own working environment rather than the shipped files.

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
