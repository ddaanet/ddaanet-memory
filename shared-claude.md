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
  Run the **gate** (`just precommit`, `just green`), never a hand-picked list
  of suites: picking them is a guess about blast radius, and removing that
  guess is what the gate is for — worst exactly when the change touches a
  shared library, which is when a subset looks most obviously sufficient. These
  gates cache on an input hash, so a re-run over an unchanged tree is free. A
  single suite while iterating mid-change is fine; the gate goes before saying
  the work is done. A slow figure recorded in a design doc (gitlore's NFR10
  puts `just precommit` at 530 s) describes work worth optimizing, not
  permission to skip it.
- **Commit `memory/` with the code**, never `.gitignore` it. When a session
  both fixes something in source and writes the memory documenting it, they
  ride **one** commit — the source diff plus the memory submodule's gitlink
  bump, not two. That does not license bundling *unrelated* pending memory work
  (a gitlink bump from an earlier, separate task) into a later fix's commit:
  bundle what corresponds, not everything pending.
- **` M memory` in the parent's `git status --porcelain` is the resting
  state**, not a dirty tree. Nobody commits inside the memory store; the
  parent's pre-commit hook records it, gates it, and stages the moved gitlink
  into whatever parent commit comes next. So a clean-tree assertion or a
  "submodules must have no uncommitted changes" rule — `ddaa:preflight` has one
  — must exclude that one path and report it as information, or it is
  unpassable until someone performs by hand a commit the release was going to
  make anyway. The exclusion covers **only** that path: untracked and modified
  files elsewhere in the parent are genuine findings. The one real precondition
  dirty memory carries is a fresh approved summary before the next parent
  commit — written after the last memory edit, since the gate compares mtimes —
  or that commit blocks and says so. That is a commit-time requirement, not a
  readiness verdict.
- **On the first turn after a compaction, check the session root** — `PWD`,
  `CLAUDE_PROJECT_DIR`, and the gitStatus block — against the work the summary
  describes. A summary says what was being done, never where the session lives,
  and the narrative is vivid where the environment block is quiet: one such
  resume ran four commits' worth of work into a repo another session owned, and
  the `/handoff` activation wipe deleted the *session's own* task file first. If
  they disagree, say so and stop; do not reconcile it by `cd`-ing. Treat a guard
  refusal, or a "modified by the user or a linter" notice on a file you did not
  touch, as evidence of a concurrent session rather than noise.
- **Report an artifact's size in bytes, not tokens**, for anything of a few
  KiB — it answers "is this material enough to care?" just as well, with no API
  key, network or cache. For a real token count consult the `claude-api` skill
  rather than memory, but note it covers only the API side: there is no exact
  offline tokenizer for Claude 3+ and the Claude-specific ones —
  `@anthropic-ai/tokenizer`, the `claude-tokenizer` Rust crate,
  `Xenova/claude-tokenizer` — are BPE approximations, rough for 3+, not just
  `tiktoken`. Cached local wrappers around `messages.count_tokens`:
  `iota-uz/cc-token` (Go, cache at `~/.cc-token/cache.json`, `--no-cache` to
  bypass) and edify's `tokens.py`/`token_cache.py`.
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
- **Two co-maintained artifacts disagree ⇒ get the drift direction from git, not
  from plausibility.** An index one-liner vs its file's frontmatter
  `description`, a spec vs code, a doc vs its implementation: do not assume which
  is fresher or authoritative. Run `git log -S/-p/--stat` on *both* surfaces and
  present concrete commits (this one changed here, that one didn't). An audit of
  exactly this found the drift **bidirectional** — sometimes the index line is
  fresh and the frontmatter stale, sometimes the reverse — so any assumed
  direction is wrong a large fraction of the time, and a sync built on the wrong
  assumption propagates staleness the wrong way. The longer text is not
  automatically the newer one.

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
- **Architecture already settled ⇒ start TDD outside-in.** With the design fixed
  (a design doc, a spec), begin from a red end-to-end test naming the
  user-facing contract and let its failure modes drive what to build next; do
  not default to bottom-up unit tests. Bottom-up units can pass while the
  integration contract is broken — they verify internal consistency, not the
  behaviour the user cares about. Outside-in tests are black-box by
  construction, so they survive refactors, the next failure names the next file
  to write (no speculative helpers), and integration risk — fixtures, stubs, env
  setup — is exercised on day 1. Backfill unit tests at the seams the e2e does
  not reach. **Fall back to bottom-up only when the architecture itself is still
  being discovered.**
- **A slow job that repeats an expensive unit needs both levers.** Parallelism
  (`bats --jobs`, `make -j`, `xargs -P`) is legitimate but never sufficient
  alone — and check the spare cores are there before making it the headline
  fix: the dev droplet resizes between 1 and 2 vCPUs to save cost when idle, so
  a fix that only pays off at 2 loses value whenever it is scaled down. Pair it
  with cutting the per-repetition cost (cache, template, memoize). The correction is additive; it does not reject parallelism. And
  **identifying the repeated unit from a subprocess or call-site count is not
  profiling** — it is a plausible model that can be wrong. Fixture-template
  caching made gitlore's highest-count helper ~3x cheaper per call and moved
  `just precommit` not at all (~595 s against a 590–613 s baseline); switching
  to `bats --timing` found two `@test`s neither model had touched and cut the
  gate ~20%. Get a per-unit timing breakdown, or wall-clock the *whole* gated
  command before and after, before implementing a targeted perf fix.
- **Constrain generated output at the artifact boundary, not mid-stream.** Let
  an agent generate loosely — scaffolding tokens, priming labels like
  `**Analysis:**` — and enforce the output constraints in a separate post-hoc
  pass with the full generated context. Scaffolding is not waste; it is the
  attention mechanism doing its work, and suppressing it at generation time
  forces silent compensating reasoning or worse output. Self-review is the cheap
  form of the second pass, a separate lint the thorough one. **Exception:
  derived artifacts** — a compressed summary of existing context, e.g. a commit
  message — need no second pass; the compression *is* the review.
- **"Reconstructable" is two categories, and only one is safe to drop.**
  Reconstructable *by the harness* — `git status`, `git log`, the files-touched
  set, the working tree — is recomputed at read time, free, correct by
  construction and fresher than any snapshot: drop it and defer. Reconstructable
  *only by model inference from a paraphrase* — the state of an in-flight task
  list after a compaction — is lossy and fails **silently**: finished work gets
  redone with nothing signalling anything went wrong. Before dropping something
  as reconstructable, name the reconstructor; if it is the model working from
  compacted context, carry it, and carry only the part inference gets wrong (for
  a task list, the *open* half — the finished half really is in `git log`, and
  keeping it is what makes a stale ledger dangerous).
- **Self-triggering skill when the condition is mechanical; command only when
  the user must initiate.** A skill whose `description` names a detectable
  condition (a hook's stderr pattern, an error string) invokes itself when the
  condition is met, so a missed trigger cannot happen; a command requires the
  user to know when to run it. Designing a new plugin entry point, ask whether a
  detectable mechanical condition should trigger it. **If the user must
  consciously decide — install, one-time setup — keep it a command.**
- **Harden where a *human* types free text; not a channel whose producer you
  control.** Before tightening how an approval or verdict signal is recognized,
  locate every point where the signal is interpreted and label each producer
  human vs controlled component. For a controlled producer (a parent agent
  instructed to send exactly `approved`/`rejected:…`) prefer an exact reserved
  sentinel, or leave the lean original alone — fuzzy NL matching there is
  theatrical. For human free text apply
  whole-word + negation-aware recognition: a hedge, a question or any negation
  is a rejection, and anything unclear is re-asked rather than proceeded on.
  Where no script sits between the human and the agent, the hardening is an
  *agent instruction*, not a literal regex; keep the detection logic in a script
  wherever there is a script to host it.

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
- **TDD red means a failed assertion, never a missing module.** "Run it, verify
  it fails (module/command missing)" is not a behavioral test — an ImportError
  proves nothing about whether the test asserts anything. Stub the SUT importable
  but inert (no-op or deliberately-wrong return), run, confirm each test fails on
  its assertion (wrong value, absent raise), then implement to green. An
  assertion that was green the first time it ran is not evidence either: it has
  never demonstrated that it discriminates, and no reviewer can recover that
  history from the finished artifact. **The inert stub is what makes a batch
  cheap to validate** — run N tests against it once and every red is
  individually visible, while a test that comes back green has identified itself
  as vacuous. What it does *not* prove is that a test detects wrongness rather
  than absence: an assertion can fail against nothing and still pass against
  both the correct implementation and the plausible-but-forbidden one it exists
  to rule out. That needs the wrong implementation run against the batch — a
  second run, not a per-test exercise. This is why the ordering matters: tests
  written before anything can pass them get their red for free, where tests
  validated against a finished implementation have to manufacture that absence
  per test.
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
- **Cut a capability in one pass, leaving nothing vestigial.** Not tests
  asserting the *absence* of the removed behaviour, not fixtures or sentinel data
  standing for the removed inputs, not a now-trivial script kept as its own file
  or subprocess — a module collapsed to a few lines is inlined into its single
  caller, not kept as a hollow shell. Half-removals force the reader to
  reconstruct what is live, and absence-guards grow stale defending behaviour
  nobody will re-add. Grep for the whole machinery — helpers, constants,
  fixtures, tests, docs, recipes, inbound references — and convert any
  absence-guard into the positive contract of what remains. **Retiring memory
  files is the same pass:** `rm` leaves every surviving `[[link]]` pointing at
  content nobody will write, so sweep the corpus for inbound links, and where a
  link carried meaning restate the rule inline rather than dropping the sentence
  (retiring 24 files cost 40 such rewrites across 30 survivors).
- **Before handling a case, establish it can occur.** Check the data model and
  the upstream source; if they cannot produce the state, write nothing for it —
  no guard, no fallback, no test. A `visited` set to make a comment walk
  "cycle-safe" defended nothing (a comment has at most one `parent_id`, so a
  cycle is unreachable from any root) and dragged in a NamedTuple to duck the
  lint it caused. The cost is real and the benefit zero — worse, a passing test
  for an unreachable state reads as verification, makes dead machinery look
  load-bearing, and freezes it against later simplification. When my human
  partner gates a request on a condition ("… *if they exist*", "if that can
  happen"), that is an instruction to go check and report what the check found,
  not a hypothetical to build for. **Contrast a case that genuinely occurs** — an
  orphan reply whose parent is missing from the corpus is real, because a
  paginated or incremental fetch can land a reply before its parent; that one
  earns its handling. Note that writing the constraint down does not stop you:
  the same session generalized a one-level comment walk to arbitrary depth with
  an h6 clamp while authoring a docstring saying replies cannot be replied to.
- **Optional means the source can genuinely omit it** — never optional "to be
  safe", "for robustness", or to make a fixture easier to write. A defaulted
  field cannot distinguish *absent because the source said so* from *absent
  because we stopped asking for it*, and in a decode chain the second is silent
  whole-corpus data loss: the struct defaults to `None`, the next layer accepts
  it, the column is quietly empty forever. Requiredness turns that into a loud
  failure at the boundary that caused it. Read the upstream schema before
  choosing: required there → required here. ghmem's `authorAssociation` was
  `str | None = None` everywhere though GraphQL types it
  `CommentAuthorAssociation!`; dropping it from the query passed every test and
  nulled the column corpus-wide, and making it required also immediately caught
  seven drifted fixtures. `mergedBy`/`mergedAt` stayed optional — an Issue node
  truly has neither. **Where a layer has a different answer from its neighbour
  that asymmetry is real information: write it down rather than flattening both
  to optional.**
- **Do not bake a strong guarantee into every call when a foreseeable caller
  must relax it.** Where durability, ordering, atomicity or validation conflicts
  with a known future need (throughput, batching), expose the guarantee as a
  separate opt-in method the caller invokes at its own checkpoints: removing one
  from a published contract is a breaking change, adding an opt-in later is not.
  ghmem's `Ledger.append` fsynced per event, which would have throttled a bulk
  fetch; it now flushes to the OS (crash-visible) and `Ledger.sync()` delegates
  to `os.fsync` when the caller wants power-loss durability.
- **Which layer owns a validation rule is set by the vocabulary the rule needs,
  not by which layer catches it first.** Before declining to add an outer check
  on duplication grounds, write the rule out: if it needs knowledge only the
  inner layer has, leave it inside (two copies of a domain rule drift); if it
  does not, it is a fact about the request's *shape* and the guard is where it
  belongs. "The heredoc must carry something" needs no verdict vocabulary; "a
  topic verdict needs a note" needs all of it. Get it wrong and a malformed
  command is accepted by the guard, then handled inconsistently downstream on an
  unrelated field. **Enforce a shape rule at *both* boundaries when a caller can
  reach the inner layer another way** — ghmem's CLI got the same empty-note
  rejection so a non-hooked caller cannot do what the hook forbids.
- **A finding that ends in "add a guard so both cannot fire" is a symptom.** Ask
  what makes the collision representable at all; the usual answer is a singleton
  resource modelled as N parallel instances whose identity lives in a **name** —
  filename, hook name, script name — so the at-most-one invariant has nowhere to
  live. Moving the identity into the data (one file whose body says which kind it
  is) makes the collision unrepresentable and collapses the per-instance
  pipeline. Treat your own "add a check" or "add a mutual check" recommendation as the
  trigger, and count
  what else is duplicated per instance: constants, validators, entry points,
  failure channels, stale-state sweeps. More than one or two and the merge is the
  finding, not the guard. Sibling-over-parameterisation is often right at two
  instances and stops paying at three, and the merge means deleting the siblings
  outright, not leaving shims.
- **Narrow a guard's accepted class to what writers need, not to what you can
  prove inert.** "Any run of non-whitespace minus the quote" and "letters, digits
  and underscores" were equally safe for a heredoc delimiter — a differential
  harness over 36 exotic delimiters (`$(…)`, backticks, `>`, `|`, `&&`, `*`,
  `\`, `"`) found no divergence from bash, because the delimiter word is
  single-quoted — but the whole guarantee rested on one excluded character and
  reading it took a paragraph of quoting reasoning rather than a look at the
  pattern. Nobody needs a `$` in a delimiter. **Run the differential test
  anyway:** it is what tells you whether you narrowed out of caution or
  necessity, and that answer belongs in the comment.
- **Parse structured formats; never hand-roll a regex over a grammar** — YAML,
  JSON, TOML, HTML — even for "just one field" ("don't use regex where you should
  really use a parser. You know, zalgo and all"). A regex breaks on the inputs
  the grammar allows but the pattern did not anticipate (quoting, escapes,
  multiline, nesting) and it fails *silently*, matching the wrong thing rather
  than erroring. If the proper parser needs a dependency, add the dependency.
  **Two things are not this:** genuine token extraction from freeform prose
  (pulling bare `#N` issue references out of markdown), where there is no grammar
  and a regex validated against real data is right; and structural splitting on
  an unambiguous delimiter (the `---` frontmatter fence) by explicit line
  handling. It is the format's *content* that must go through the parser.
- **No `2>/dev/null`** unless failure is normal and the message expected — and
  even there, prefer surfacing the message unless it matches expectation. A
  blanket redirect silences one anticipated message and every unanticipated one
  with it: two gitlore defects hid behind exactly that (both `git push` sites
  assumed failure meant unreachable-or-divergence, so a protected branch or
  `pre-receive` decline started a merge resolution that could not help, with
  git's explanation discarded; a suppressed `ls -A` let an unreadable directory
  read as "empty"). In order: **(1) guard the expected failure away** so what
  remains on stderr is real (`[ -f "$file" ]` before a reader, `[ -e path/.git ]`
  before a submodule `git -C`, `rev-parse -q --verify MERGE_HEAD` before `merge
  --abort`); **(2) capture and match** when several failure modes are treated
  differently — `err=$(cmd 2>&1)` then `case "$err"` on a verified
  discriminator; **(3) keep the redirect only when provoking the error IS the
  mechanism** (a write probe, `python3 -c 'import yaml'` feature detection) or
  when every failure mode is normal and answered by a safe default — and write
  the reason inline. Never replace git's message with a guess: worktree-remove
  appended "(locked or uncommitted)" to every failure.
- **No branch, fallback or diagnostic narrative for a transition between
  versions of tooling you also control** — a plugin update that has not re-pinned
  its config key, an old on-disk layout, a half-migrated path. Fix forward:
  "don't write a special case for the gitlore transition, it has a user base of
  one." With one user the transition window is a single restart, the branch
  outlives the case, and its explanatory prose is wrong the moment the transition
  ends. When a message starts enumerating causes ("installed before this session
  / updated since / the pinned path moved"), collapse it to one fact plus one
  recovery act. **Reserve real migration handling for boundaries with users you
  do not control**, and note that read-only *liveness* detection of a foreign
  tool's state is a different thing that still applies.
- **Detecting that a *foreign* tool is mid-run takes more than
  `[ -f "$known/path" ]`.** Require the current layout **and** a positive
  identity marker inside the file (a header line naming what it belongs to), and
  define which one wins when several match. Two independent failure modes make
  presence-at-a-path a false positive: the layout moves — the other tool
  relocates its file and often declares the old path a stray, so the one path
  tested becomes the one guaranteed *not* live — and nothing cleans up, so a file
  left by an abandoned run survives forever, more so in a git-ignored tree where
  `git status` never shows it. handoff hit both at once: a leftover
  `.superpowers/sdd/progress.md` made a session running no SDD suppress its own
  `handoff-todo.md` *and* append its findings to a ledger headed with a
  different plan's name. Pick most-recently-modified among several matches
  (alphabetical glob order is not a liveness signal), make the near-miss fail
  **open** (no detection → do the normal thing), verify layout and marker against
  the other tool's own source rather than its docs, keep the detector read-only
  (another workflow's stale file is never yours to delete), and keep one registry
  function owning the layout with every consumer interpolating what it prints.
- **Ground every heuristic constant in data, and never let a pending constant
  license a placeholder implementation.** A scoring formula, weight or threshold
  is derived from analysis of the data it will run on — percentiles,
  known-ground-truth calibration — with the analysis recorded; "always ground
  heuristic formulas in data analysis" was the correction to a hip-shot heat
  formula and its 3-PRs/10-commits thresholds. Keep them tunable, tuned from
  evidence. Meanwhile **weights are parameters: bind them into the query.** Only
  a change in the model's SHAPE (nonlinear score, rank fusion, a normalization
  pass) forces a rewrite — "the numbers aren't final" never does, and neither
  `--limit` nor "calibrate first, then optimize" excuses a per-item Python loop
  that is a reimplemented `GROUP BY` — check whether the aggregate is
  expressible in SQL before writing the loop, and when a plan defers an
  optimization "unless calibration shows it matters", ask whether the deferral
  is really about the model's shape; if it is not, build it properly now,
  because the calibration consumer is the one that most needs it done right. Where the consumer of an output is
  analysis, emit a data format (CSV with a header, JSONL), not a dense human line
  to be regex'd apart. **And judge a weight by fitting the whole vector, never a
  per-metric sort:** sorting a labelled sample one feature at a time measures
  *marginal* discrimination while a weight is *conditional*, and the gap gets
  useful features deleted. ghmem's `related_items` sorted signal at exactly the
  41% base rate at n=54 while carrying the heaviest weight in `HEAT_WEIGHTS`
  (3.0), and it remains the weakest lone ranker (AUC 0.595); at n=198 its fitted
  coefficient was +0.243, second largest, and zeroing it cost corpus-weighted AUC
  (0.818 against 0.843). Both are true at once. Before cutting a weight, check what dropping it costs
  the full score. When a sample is stratified on the composite, its
  single-metric-versus-composite comparison is handicapped by construction —
  treat any "the part beats the whole" result from it as an artifact until a
  differently drawn sample reproduces it.
- **A justfile's shared `bash_prolog` is for common foundations only:** a
  function earns its place by having two or more call sites, and anything called
  once is inlined into the recipe that calls it — including the tempting
  `run-<recipe>` wrapper holding a whole body. The prolog is copied between
  projects, so every function in it is inherited and has to be understood by the
  next one. Corollaries: early exit is `exit` **in the recipe**, never `return`
  from a helper; setup like `sync` is called once at **recipe toplevel**, not
  from inside a helper several recipes reach (or a dependent recipe calls it
  twice); and a helper written for cross-project reuse is the one legitimate
  exception — parameterize it (`check-sentinel <name>` → `tmp/.<name>-sentinel`)
  and say so.
- **Never `uv run`, never `uvx`, under Claude Code — `uv` is for `uv sync` and
  `uv tool install`.** Every `uv run` touches `~/.cache/uv`, which the Bash sandbox blocks,
  and threading that through settings and permissions per project is more
  friction than it buys. Rely on direnv to activate the venv (`.envrc` exports
  `VIRTUAL_ENV=$PWD/.venv` and prepends `$VIRTUAL_ENV/bin` to `PATH`), and have
  recipes call tools by bare name — `pytest`, `ruff`, `mypy` — never
  `./.venv/bin/foo`. A tool with no venv-compatible release (`docformatter` on
  Python 3.14) goes in via `uv tool install` and is still called bare, off
  `~/.local/bin`.
- **A quiet gate signs off with the list of checks that passed**, accumulated as they
  run — never a bare "OK" or "Green". Silence is ambiguous: a check that printed
  nothing looks exactly like one that never ran. A recap resolves it because a
  missing check is visibly missing. Failure stays terse.

## Writing docs, plans and specs

- **A plan doc is as small as the work.** When the work is concrete ("edit N
  files + run M commands"), padding with architecture diagrams,
  lessons-learned openers, dogfood narratives and self-review checklists feels
  productive and only adds review burden: a 700-line packaging plan reduced to
  7 executable steps ("why 700 when 7 would do?"). List the actual executable
  steps first; under ~10 commands with no genuine design decisions, skip the
  spec-style template and write a ≤30-line checklist of steps, open decisions
  and scope notes. **Reserve full spec docs for plans with non-trivial design
  decisions worth recording across sessions** — plans carrying real FRs, NFRs
  and alternative analysis fit the template; packaging plans do not.
- **Never hand a long document over with "look this over."** Large docs hide
  errors in volume and reviewers — human or LLM — skim instead of inspect
  ("documents are too large for eyeballs"; a ~680-line plan drew exactly one
  annotation). Do a fresh-eyes pass yourself first and flag the specific
  high-risk sections — concrete shell snippets, cross-repo paths, file lists,
  anything that must be checked against the world rather than read — as "lines
  X–Y do Z, worth verifying". The brainstorming skill's "spec self-review" step
  exists for exactly this; do it with rigor rather than ticking the boxes. For doc edits prefer a revision-only diff
  (`revdiff HEAD~1`) over a cumulative `HEAD~N`, and split structurally where
  you can: spec in its own commit, implementation plan in another.
- **Meet a line-count cap by wrapping honestly, never by cramming.** Hard-wrap
  prose at the repo's ~72-col convention *first*, then measure; if it still
  exceeds, split on a clean seam. Long unwrapped 150–300-char lines game the
  number while destroying the reviewability the cap protects ("you are clearly
  cheating on the line count metric"). Preserve all information across a split —
  duplicated front matter (constraints, shared types) is fine, dropped content is
  not. **Cramming has two further forms:** a full interface conveyed as a run-on
  paragraph (semicolon-chained methods, comma-run dataclass lists) — give each
  method, dataclass and exception its own bullet with its full signature and
  return type, which also surfaces elided return types; and, on *source* files,
  shaving docstrings to fit — prose is the cheapest thing to cut and the first
  thing a later reader needs, and a file parked at exactly its cap has stopped
  emitting the signal the cap exists to emit. If trimming is the only way to fit,
  say so out loud and put the split in front of the human as a decision. **GitHub
  bodies are the opposite case** — they render every newline as a break, so no
  hard wrap there. **And the cap is not absolute:** it is a reader-context
  budget, and a split imposes its own cost (re-reading several docs, chasing
  cross-references), so accept a bounded overage on one cohesive doc when the
  split would be *more* human effort — a validated plan ~40 lines over 400 was
  deliberately kept whole. That tempers the "always split when over" reflex only;
  the honest-wrap discipline still holds.
- **A single task's contract running past ~400 lines is the signal to split that
  task.** Contract size predicts implementation size — a ~400-line contract
  produced a 1385-line implementation — and catching the overshoot at
  spec-review time is cheap where discovering it at PR-slicing time is not.
  Weight contract length as a predictor in the update-vs-split decision rather
  than judging the contract's size in isolation. The 400-line cap on
  human-reviewed artifacts is a **soft** reviewability guideline: do not block a
  commit or agonize over exceeding it in local self-review.
- **A spec never references its own previous versions.** Scrub "was confusingly
  similar", "the earlier constraint is dropped", "(good) not (bad)" comparisons
  against a prior draft: implementers do not know and do not care about earlier
  versions, and self-referential history is noise. State the surviving design
  positively. **KEEP rejections of real design alternatives that stand on their
  own** ("shelling out to `gh api graphql` was rejected: …") — those are
  decisions, not draft archaeology. Introduce private or opaque references
  (sibling repos, personal tooling) on first use instead of assuming the reader
  knows them.
- **A lasting guideline stays general; the audit is the repo-specific
  artifact.** Asked to research best practices and produce a reference meant to
  amortize across releases, do not ground it in the current repo or frame it as
  an audit input — a research prompt scoped to "ground in THIS project" leaked
  ~19 "this repo does X" intrusions plus Repo-notes and Review-log sections into
  a doc meant to be durable, and repo commentary makes a general reference read
  as a stale one-off. Split the two: the **guideline** is general DO/AVOID rules
  with neutral illustrative examples, version-grounded but not repo-grounded; the
  **audit** holds the repo-specific findings and is point-in-time. In a
  multi-stage research→review→audit pipeline only the audit stage is
  repo-specific. Confirm the intended scope before delegating the research.
- **A style brief that states a goal without a method delegates badly.** Given
  "split em-dash pile-ups" and "unpack telegraphic fragments", subagents
  converted every em-dash and explanatory colon into a full stop, and a
  corrective wave left quoted regressions untouched while over-reverting others.
  The judgement the work needs — *is this edit clearer, or just longer?* — was
  never briefed, and terse voice reads as "crammed" to an agent told to uncram.
  Brief the *how*: a worked before/after example of one unpacking, and an
  explicit "keep the em-dash, split the ideas". Where that cannot be written
  down, delegate the mechanical verifiable half and keep the rest — structural
  work with a machine-checkable exit condition ("`check_line_limits.sh` must exit
  0") delegates fine. Read the full diff before reporting it done. **Two limits
  on this evidence:** "limp" was a self-assessment, never a confirmed quality
  drop, and the cause was brief specification, not model tier — do not cite it as
  a prior that prose degrades down-tier.

## Skills and agent definitions

- **No rule or anti-pattern for a failure nobody has seen.** An anti-pattern
  list is a record of defects that happened; plausible-sounding padding dilutes
  the real entries and costs context on every load. Before adding one, name the
  incident — no incident, no entry. A sharper variant, seen once and recorded as one instance rather than a rate
  (whether it generalizes is unknown, but it is cheap to avoid): an invented
  guardrail around a new decision point, phrased as a prohibition, forbade
  exactly what the common request wanted ("when the user says 'handoff and
  commit', forbidding git commands prevents the expected behaviour"). **When a
  rule must exist, state it positively in the step where it acts** ("the commit
  lands before this file is written") rather than as a prohibition on the agent
  — a positive ordering constrains sequence without forbidding the action.
- **A skill's `description:` opens with what the skill does**, then the
  triggers / "Use when…" clause, then optional exclusions. It is shown verbatim
  in the Claude Code TUI skill list, so a trigger-first description reads as
  noise and the user cannot tell what the skill is for at a glance. Drop
  overspecific or stale implementation detail from the tail — mechanics are not
  a useful TUI summary. Applies to every skill in a plugin, not just the
  headline one. **Model guidance does not belong in a skill body at all:** which
  model runs the work is an agent-frontmatter concern, so "run this on Sonnet"
  in a skill states something it cannot enforce.
- **The imperative-form rule for skill bodies targets instruction subjects**,
  not every pronoun. `plugin-dev:skill-development`'s prohibited examples are all
  second person as the subject of an instruction — `You should create…`,
  `You need to configure…`. Reflexive intensifiers (`do not verify it
  yourself`), possessives naming a referent (`your post-compaction self`) and
  passive objects (`submitted for you`) are not what it targets, and a bare
  `grep -w you` reports all of them identically — in handoff's two skill bodies,
  11 hits contained exactly 1 true violation. Classify each hit before
  rewriting: instruction-subject (rewrite verb-first), load-bearing intensifier
  (replace the word rather than delete it — `run it yourself` → `run it
  directly` keeps the agent-vs-machinery contrast), removable intensifier
  (delete), or a referent with no good third-person label (keep). handoff
  deliberately keeps `your post-compaction self` in
  `skills/precompact/SKILL.md` — it asserts the identity continuity that
  motivates writing the file at all, which `the post-compaction agent` drops.
  Do not "fix" it on a later grep sweep.

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
