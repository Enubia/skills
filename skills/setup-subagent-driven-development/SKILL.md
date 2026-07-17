---
name: setup-subagent-driven-development
description: Configure one repo to use the subagent-driven-development skill — document the execution step in CLAUDE.md/AGENTS.md, record where tasks come from, and choose feature-branch or worktree isolation. Run once per repo; works standalone, without the rest of the engineering skills.
disable-model-invocation: true
---

# Setup Subagent-Driven Development

Make a repo `subagent-driven-development`-aware: record, where agents will look for it, that SDD is this repo's in-session execution path, where its task briefs come from, and whether each run uses a feature branch in the current worktree or a separate worktree. Every run creates a new work branch from the branch active when SDD is invoked.

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write. Be idempotent: update an existing block in place, never append a duplicate.

## Prerequisite check

Confirm the `subagent-driven-development` skill is actually installed (look for `subagent-driven-development` in the available skills, or `~/.claude/skills/subagent-driven-development`). If it isn't, tell the user to install it first — this skill only configures a repo to use it; it does not install it.

## Process

### 1. Explore

Read the repo's starting state; don't assume:

- `git remote -v`, current branch, `git worktree list` — is there a remote, and does the repo already use worktrees?
- `CLAUDE.md` and `AGENTS.md` at the repo root — does either exist? Does `CLAUDE.md` delegate to or reference `AGENTS.md`? Is there already an `## Agent skills` section, and does it already mention execution / SDD?
- `.scratch/` — are there issue dirs (`<feature>/issues/`, or a ticket-prefixed variant like `<TICKET>-<slug>/issues/`)? Sign that an issue-tracker convention is already in use and issues can be the task source.
- `docs/agents/` — does prior setup output exist (`issue-tracker.md`, `execution.md`, etc.)? Does it already record a task source or workspace mode?
- `CONTEXT.md` and `docs/adr/` — present? Reviewers and implementers will read them if so (optional, not required).

### 2. Present findings and ask

Summarise what's present and what's missing in two or three lines. Then ask one batched question with two decisions, each with a detected default. Assume the user may not know the terms; lead each decision with a one-line explainer.

**Task source.**

> Explainer: SDD executes one task per implementer subagent. It needs to know what a "task" is in this repo — a ready-made issue file it hands over verbatim, or a section of a larger plan file it extracts. SDD supports both; this just sets the default wording so the docs match how you actually work here.

- **Agent-ready issue files** — each task is a markdown file (e.g. `.scratch/<feature>/issues/<NN>-<slug>.md`). The issue file is the implementer's brief verbatim. Default this if issue dirs already exist under `.scratch/`.
- **A plan file** — one document with `## Task N` headings; SDD extracts each task with its `scripts/task-brief`. Default this if there's no issue convention.
- **Both** — accept either. Safe default when unsure.

**Workspace mode.**

> Explainer: every SDD run records the branch active at invocation, then creates a fresh work branch from that exact commit. The workspace mode decides where that new branch is checked out.

- **Feature branch** — create and check out the new branch in the current worktree. Default this unless the repo already uses worktrees or its instructions favor them.
- **Worktree** — create the new branch in a separate worktree, leaving the current checkout on the base branch. Default this when existing instructions, tooling, or `git worktree list` show that convention.

Always ask; the detected default is a recommendation, not an inferred answer. Treat the branch active when SDD is invoked as the fixed base, so setup needs no branch-base choice. If `CONTEXT.md`/ADRs exist, just note in the doc that reviewers should read them.

### 3. Confirm and write

Show the user a draft of the `## Agent skills` block addition and the `docs/agents/execution.md` contents, including the selected task source and workspace mode. Let them edit before writing.

**Pick the file to edit** (use the active repo instructions file; don't always write `CLAUDE.md`):

- If `CLAUDE.md` exists and references `AGENTS.md`, edit the referenced `AGENTS.md`. If the referenced file is missing, ask before creating it and mention that `CLAUDE.md` already points there.
- Else if `CLAUDE.md` exists and does not reference `AGENTS.md`, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which to create — don't pick for them.

Only edit `CLAUDE.md` directly when it does not delegate SDD-relevant instructions to `AGENTS.md`. Never edit both files for the same setup. If an `## Agent skills` block already exists, add or update the `### Executing work` subsection in place; leave the other subsections untouched.

**The subsection to add** (fill the bracketed parts from the task-source answer):

```markdown
### Executing work

Once tasks are ready, execute them in-session with the
`subagent-driven-development` skill: one fresh implementer subagent per task
(building test-first via `tdd`), a spec+quality review after each, then a
whole-branch review before merge. Task source: [issue files under
`.scratch/<feature>/issues/` | a plan file with `## Task N` headings | either].
Workspace mode: [feature branch in the current worktree | separate worktree].
Every run creates a new work branch from the branch active at invocation.
SDD's transient run artifacts live in self-ignored `.scratch/sdd/` — never
committed. See `docs/agents/execution.md`.
```

Then write `docs/agents/execution.md` from the seed in this skill folder ([execution-doc.md](execution-doc.md)), filling the task source and workspace mode and noting whether `CONTEXT.md`/ADRs are present for reviewers to read.

### 4. Done

Tell the user setup is complete: in this repo, "execute the plan / these issues" will now route to `subagent-driven-development`, and agents reading the edited instructions file (`CLAUDE.md` or `AGENTS.md`) will know the task source and workspace mode. They can edit `docs/agents/execution.md` directly later; re-run this skill to change either choice.
