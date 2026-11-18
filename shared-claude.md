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
  now ship as installed plugin skills, and where both exist the skill is the maintained
  copy. Be selective: a skill body costs more than a memory file, so load what
  names the moment you are in, not everything adjacent.
- **My human partner owns branching.** Never create or switch branches — or
  worktrees — unprompted, even when a workflow skill says to start feature
  work on a fresh one. Work on whatever is checked out. Merging onto a base they named is
  executing their instruction, not originating a switch, and the release
  sequence in `edify:release` is standing consent to the switches it names.
  A plan my human partner approved that names a separate branch is the prompt
  to create it.
- **Each content commit is reviewed as it lands.** Every commit that generates,
  modifies or deletes content gets a delegated `/code-review medium` right after
  it lands, its high and medium findings fixed in a commit of their own and its
  lows declined; doc drift is fixed without a re-review. Load `edify:review`
  before triaging findings. A release is `edify:release`; in claude-plugin-dev
  and in a plugin whose `plugin-dev/VERSION` is 0.16.0 or later everything runs
  on `dev` and the release moves `main`. Never ask whether to move `main`.
- **Never `--no-verify`.** Let every hook run and surface a block instead of
  bypassing it; the parent repo's pre-push hook is what publishes memory with
  the push, so skipping it strands the memory commits the push records.
- **Phantom dotfiles are sandbox masks.** Zero-byte or char-device dotfiles
  and `.claude/*` entries showing untracked in a repo root, or in the directory
  a sandboxed call ran from, are masks the command sandbox leaves and say
  nothing about the repo: never delete, commit or report them, and never act on
  a slash command's `git status` block without re-running it unsandboxed.
  Mechanism: `sandbox-lies:sandbox-lies`.
- **Run the repo's precommit gate before committing**, unprompted — it is a
  required step, not a suggestion to float. In micro that is `just precommit`.
- **Refer to the user as "my human partner"**, never by name, in anything I
  author as myself: memory files, gate summaries, handoffs, internal docs.
  Content written *in their name* for an external audience follows the
  project's own voice instead, and git authorship carries their public
  identity by design.
- **A summary of a change points at the diff, it does not narrate it.** For
  a simple, self-contained change, name the file (a line only if needed) and
  say why; anything that repeats the diff in other words goes. Applies to
  commit messages and memory commit summaries.
- **A commit carries everything that belongs to its change.** Related
  working-tree changes stay in: never drop a file from a commit silently —
  name the exclusion and let my human partner decide. A memory note rides the
  commit of the change it documents, never one carrying unrelated pending
  memory work, and a `.claude/settings.json` change rides the feature it
  enables.
- **A file split travels alone.** Moving part of a file into a new one, for a
  size cap or a seam, is a commit of its own: the move and the reference
  updates it forces, with no reflow, rewording or content edit riding along,
  sequenced before or after the edit that motivated it. An edit hidden among
  hundreds of moved lines is invisible in review.
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
- **Dispatch through `edify:dispatch`.** An edit that touches three or more
  files, or a sweep of any size, is dispatched rather than done on the main
  thread. Load `edify:dispatch` before any Agent call. Leave the Agent tool's
  `name` unset.
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

- **A design question opens with the problem, in terms they can check.** One at a
  time: what they do, what happens, why that is wrong, and only then a proposal.
  Internal predicates, helper names and probe ids are not a problem statement, and
  a problem that cannot be said this way is doubted before it is asked. My human
  partner, handed five compressed questions with ready defaults: "They seem to have
  sensible answers, but experience tells me that 'sensible-sounding solution to
  gibberish problem' is where AI bugs and overengineering goes to reproduce."
- **"next?" asks what is next; it does not say to start it.** Answer with the
  next step and wait. On a plugin repo with release-worthy commits on `dev`
  not yet on `main`, that answer is the full release cascade, not a bare
  push: installs resolve through the marketplace, so a push alone reaches no
  user, and release subsumes push. Name push-only when the commits are
  deliberately not release-worthy (docs-only, in-flight work).
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
- **A brief's recommendation is input, not a decision.** Its preference order
  and constraints are another agent's assumptions. Verify the recommendation
  cheaply, surface the choice and get my human partner's call before writing
  code; keep non-contingent work going meanwhile.

## Deciding and planning

- **Take decisions as late as possible, but not later.** Deferring buys information;
  past the deadline, momentum and accreted code make the decision by default. Name an
  unforced question as open, state what evidence would settle it, and do not write
  code that presupposes the unsettled answer. Write a plan right before executing it,
  and plan only the iteration in front of you — pre-writing is fine only when asked.
- **Always-on context takes prohibitions; procedures defer to their trigger.** A
  guardrail against something the agent would otherwise do unprompted must be
  front-loaded — nothing downstream discloses it in time. **A gate that emits
  its own instructions is invoked, not pre-satisfied** — its refusal *is* the
  report, carrying the format, the path and the next command, so triggering it
  costs one call while reconstructing it by hand costs several and still gets
  details wrong. That holds for gates that are cheap, reversible and
  self-describing; it is not a licence to fire a destructive command and read
  the error.

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
- **20000 bytes caps any artifact a reader takes in at one go, 10000 a
  `SKILL.md` hub** — as `wc -c` counts, hard for design docs and implementation,
  soft for plans. Over the cap, split on a when-needed seam; never shave prose
  to fit. A `size.exempt` entry is never the alternative to the split; it may
  only hold a split already planned, with its end named in the entry's reason.
  `check-size` enforces it; the argument is in `craft:plan-contracts`.
- **A format with a grammar goes through its parser, never a regex** — YAML,
  JSON, TOML, HTML, even for "just one field"; a regex over a grammar matches
  the wrong thing silently on quoting, escapes or nesting. If the parser needs
  a dependency, add it. Regex is for token extraction from freeform prose,
  validated against real data, and splitting on an unambiguous delimiter line
  (the `---` frontmatter fence) by explicit line handling is fine.
- **Write no code that is not needed.** Before adding an existence check, an
  "only when" branch or a dedup pass, state what goes wrong without it; if the
  unguarded form is inert (a PATH entry naming a missing directory), drop the
  guard, and propose removing such guards in review. A guard against a
  harmless state usually also blocks a useful one, so "nothing produces that
  state" does not defend one that costs a branch or a restart.
- **A quiet gate signs off with the list of checks that passed**, accumulated as they
  run — never a bare "OK" or "Green". Silence is ambiguous: a check that printed
  nothing looks exactly like one that never ran. A recap resolves it because a
  missing check is visibly missing. Failure stays terse.
- **Whitespace safety is essential — never a nit.** Treat splitting on whitespace as
  a defect on sight. Severity claims from reading are guesses: run it against a
  spaced input first. Default to NUL-delimited forms (`git config -z --get-regexp`,
  `find -print0`, `read -r -d ''`). State a residual bound in a comment rather than
  implying full coverage.

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
