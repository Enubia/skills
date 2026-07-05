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

## How a run goes

1. Work on a feature branch or worktree — never `main`/`master` without consent.
2. Per task: dispatch a fresh implementer subagent (builds test-first via the
   `tdd` skill) → generate a diff package → dispatch a task reviewer (spec
   compliance + code quality) → loop fixes until clean → record the task in the
   progress ledger.
3. After all tasks: one whole-branch correctness/spec review, then finish
   (offer the maintainability pass, then merge/PR per repo conventions).

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
