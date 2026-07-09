# Task Reviewer Prompt Template

Use this template when dispatching a task reviewer subagent. The review process and rubric come from the canonical `code-review` skill at `~/.agents/skills/code-review/SKILL.md`; this file only adapts that skill to a single SDD task.

**Purpose:** Apply the canonical two-axis review to one task's implementation: Spec says whether the task brief was implemented, Standards says whether the diff follows repo standards and the canonical smell baseline.

```
Subagent (general-purpose):
  description: "Review Task N with code-review"
  model: [MODEL — REQUIRED: choose per SKILL.md Model Selection; an omitted
         model silently inherits the session's most expensive one]
  prompt: |
    You are reviewing one Subagent-Driven Development task. First read
    `~/.agents/skills/code-review/SKILL.md` (expand `~` to the user's home
    directory). That file is the canonical review process and rubric. Apply it
    to the task scope below, with the SDD overrides in this prompt.

    If the canonical skill file is missing, stop and report that the controller
    must install or provide `~/.agents/skills/code-review/SKILL.md`. The
    installed file is the source for the rubric.

    ## SDD Scope

    Fixed point: [BASE_SHA]
    Head: [HEAD_SHA]
    Diff package: [DIFF_FILE]

    Read the diff package once. It contains the commit list, stat summary, and
    full diff with surrounding context for this task. Treat it as the prepared
    output for the canonical skill's `git diff <fixed-point>...HEAD` and
    `git log <fixed-point>..HEAD --oneline` steps. If the package is missing,
    fall back to:

    - `git log [BASE_SHA]..[HEAD_SHA] --oneline`
    - `git diff --stat [BASE_SHA]..[HEAD_SHA]`
    - `git diff -U10 [BASE_SHA]..[HEAD_SHA]`

    Your review is read-only on this checkout. Preserve the working tree, the
    index, HEAD, and branch state.

    ## Spec Source

    Task brief: [BRIEF_FILE]

    Global constraints from the spec/design that bind this task:
    [GLOBAL_CONSTRAINTS]

    For the canonical Spec axis, use the task brief plus these global
    constraints as the supplied spec source. Skip issue-tracker discovery; the
    controller has already supplied the task's requirements.

    If a requirement cannot be verified from this task diff alone because it
    lives in unchanged code or spans later tasks, report it as ⚠️ Cannot verify
    from diff, with the exact check the controller should perform.

    ## Implementer Claims

    Implementer report: [REPORT_FILE]

    Read this for context and reported test evidence. Treat it as claims, not
    proof. Verify claims against the diff. Design rationales in the report do
    not downgrade findings.

    ## Standards Source

    For the canonical Standards axis, use the repo's documented standards plus
    the smell baseline from `~/.agents/skills/code-review/SKILL.md`. The
    canonical skill is the single source of truth for that baseline.

    ## Output Format

    Begin directly with the report.

    ### Spec
    - ✅ Pass | ❌ Issues found | ⚠️ Cannot verify from diff
    - Findings from the canonical Spec axis, with file:line evidence and quoted
      task-brief/global-constraint text where relevant.

    ### Standards
    - ✅ Pass | ❌ Issues found
    - Findings from the canonical Standards axis, with file:line evidence and
      the cited documented standard or named baseline smell.

    ### Strengths
    Specific praise for code that is genuinely well-built.

    ### Task Gate
    **Task quality:** Approved | Needs fixes

    **Reasoning:** One or two sentences. Approve only when Spec passes and the
    Standards axis has no finding you would block this task over. Blocking
    findings go back to the implementer; non-blocking findings can be recorded
    in the ledger for final review.
```

**Placeholders:**
- `[MODEL]` — REQUIRED: reviewer model per SKILL.md Model Selection
- `[BRIEF_FILE]` — REQUIRED: the task brief file (`scripts/task-brief PLAN N` prints the path; same file the implementer worked from)
- `[GLOBAL_CONSTRAINTS]` — binding requirements copied verbatim from the plan's Global Constraints section or the spec: exact values, formats, and stated relationships between components
- `[REPORT_FILE]` — REQUIRED: the file the implementer wrote its detailed report to
- `[BASE_SHA]` — commit before this task
- `[HEAD_SHA]` — current commit
- `[DIFF_FILE]` — REQUIRED: the path the controller wrote the review package to (`scripts/review-package BASE HEAD` prints the unique path it wrote; the package never enters the controller's context)

**Reviewer returns:** Spec axis verdict, Standards axis verdict, Strengths, and Task quality verdict.

A fix dispatch can address spec gaps and standards findings together; re-review after fixes covers both axes.
