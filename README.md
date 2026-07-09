# Skills

Agent skills for [Claude Code](https://claude.com/claude-code). Each skill lives in `skills/<name>/` with a `SKILL.md` entrypoint plus any bundled prompts and scripts.

## Skills

### [`quality-review`](skills/quality-review/SKILL.md)

A user-invoked thermonuclear maintainability review for structural simplicity, abstraction quality, file-size growth, and spaghetti branching. It produces a single-file HTML report with a clear approval call and the highest-leverage fix.

### [`subagent-driven-development`](skills/subagent-driven-development/SKILL.md)

An in-session execution engine for a plan or a set of agent-ready issues. It dispatches a **fresh implementer subagent per task** (building test-first via `tdd`), runs the canonical **code-review Standards/Spec axes** after each task, then a **whole-branch review** before merge — all without leaving the current session.

Each subagent gets an isolated, hand-crafted context instead of inheriting the session history, which keeps them focused and preserves the controller's context for coordination. Reports and review packages are handed over as files (under self-ignored `.scratch/sdd/`) rather than pasted into context, and a durable progress ledger survives compaction so completed tasks are never re-run.

Use it when executing an implementation plan or the issues under a ticket whose tasks are mostly independent. Planning happens upstream (`grilling` → `to-issues`); this skill only executes.

### [`setup-subagent-driven-development`](skills/setup-subagent-driven-development/SKILL.md)

A one-time, per-repo configurator for the skill above. It documents SDD as the repo's in-session execution path in `CLAUDE.md`/`AGENTS.md`, records where task briefs come from (issue files vs. a plan file), and writes `docs/agents/execution.md`. Idempotent and standalone — it works without the rest of the engineering skills installed.

## Installing

Install with the [`skills`](https://github.com/vercel-labs/skills) CLI — no clone required:

```sh
npx skills add Enubia/skills                                    # pick from a list
npx skills add Enubia/skills --all                              # install every skill
npx skills add Enubia/skills --skill subagent-driven-development  # a specific one
```

Claude Code discovers each skill by the `name` in its frontmatter.
