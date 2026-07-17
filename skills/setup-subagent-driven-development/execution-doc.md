# Executing work: Subagent-Driven Development

This repo executes implementation work in-session with the
`subagent-driven-development` (SDD) skill. SDD is the *execution engine* — it
runs an already-agreed plan; it does not plan. Sharpen the design and break it
into tasks first, then hand the tasks to SDD.

## Task source

<!-- setup fills ONE of these -->

- **Issue files.** Each task is a markdown file under
  `.scratch/<feature>/issues/<NN>-<slug>.md`. The issue file is the
  implementer's brief verbatim — SDD passes its path straight to the subagent.
  Execute them in numbered order, respecting stated dependencies.
- **Plan file.** One document with `## Task N` headings. SDD extracts each
  task's brief with its bundled `scripts/task-brief PLAN_FILE N`.
- **Either.** Both shapes work; use whichever a given piece of work produced.

## Branch isolation

<!-- setup keeps ONE workspace mode -->

- **Feature branch.** At invocation, record the active branch and its exact
  `HEAD`, then create and check out a new work branch from that commit in the
  current worktree.
- **Worktree.** At invocation, record the active branch and its exact `HEAD`,
  then create a new work branch from that commit in a separate worktree. Leave
  the original checkout on the base branch.

The branch active when SDD is invoked is always the base, even when it is itself
a feature branch. Capture it before repository updates. Use the recorded base
commit for the final diff and the recorded base branch as the merge or PR
target.

## How a run goes

1. Establish the configured workspace and verify that the new work branch is
   checked out from the recorded base commit.
2. Per task: dispatch a fresh implementer subagent (builds test-first via the
   `tdd` skill) → generate a diff package → dispatch a task reviewer (spec
   compliance + code quality) → loop fixes until clean → record the task in the
   progress ledger.
3. After all tasks: one whole-branch correctness/spec review against the
   recorded base commit, then finish (offer the maintainability pass, then
   merge/PR to the recorded base branch per repo conventions).

## Workspace

SDD's transient run artifacts — implementer reports, review packages, and the
`progress.md` ledger — live under `.scratch/sdd/`. That folder self-ignores, so
it never shows in `git status` or gets committed. It is the recovery map after
a context compaction: trust the ledger and `git log` over recollection. Don't
confuse it with the tracked issue directories.

## Domain docs

<!-- setup notes whether these exist -->

If this repo has a `CONTEXT.md` (domain glossary) and `docs/adr/`, reviewers and
implementers read them for the area they touch and use that vocabulary — and do
not re-litigate a decision an ADR already settled.
