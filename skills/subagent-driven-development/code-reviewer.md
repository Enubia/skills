# Code Reviewer Prompt Template

Use this template when dispatching the final whole-branch reviewer subagent. The review process and rubric come from the canonical `code-review` skill at `~/.agents/skills/code-review/SKILL.md`; this file only supplies the SDD branch scope and output gate.

**Purpose:** Apply the canonical two-axis review to the whole branch before merge. Spec checks plan/requirement compliance; Standards checks documented repo standards plus the canonical smell baseline. The stricter code-judo audit remains the separate user-invoked `quality-review` pass.

```
Subagent (general-purpose):
  description: "Review branch with code-review"
  model: [MODEL — REQUIRED: the final whole-branch review takes the most
         capable tier per SKILL.md Model Selection; an omitted model silently
         inherits the session's, which may not be it]
  prompt: |
    You are reviewing the completed branch before merge. First read
    `~/.agents/skills/code-review/SKILL.md` (expand `~` to the user's home
    directory). That file is the canonical review process and rubric. Apply it
    to the branch scope below, with the SDD overrides in this prompt.

    If the canonical skill file is missing, stop and report that the controller
    must install or provide `~/.agents/skills/code-review/SKILL.md`. The
    installed file is the source for the rubric.

    ## Branch Scope

    What was implemented:
    [DESCRIPTION]

    Fixed point: [BASE_SHA] (the exact base commit recorded when SDD was invoked)
    Head: [HEAD_SHA]
    Diff package: [DIFF_FILE]

    Read the diff package once. It contains the commit list, stat summary, and
    full diff with surrounding context for the whole branch. Treat it as the
    prepared output for the canonical skill's `git diff <fixed-point>...HEAD`
    and `git log <fixed-point>..HEAD --oneline` steps. If the package is
    missing, fall back to:

    - `git log [BASE_SHA]..[HEAD_SHA] --oneline`
    - `git diff --stat [BASE_SHA]..[HEAD_SHA]`
    - `git diff -U10 [BASE_SHA]..[HEAD_SHA]`

    Your review is read-only on this checkout. Preserve the working tree, the
    index, HEAD, and branch state. If you need a different revision checked
    out, use a separate temporary worktree.

    ## Spec Source

    Requirements / plan:
    [PLAN_OR_REQUIREMENTS]

    For the canonical Spec axis, use this as the supplied spec source. If it is
    a path, read it. If it is inline text, use it directly. Skip issue-tracker
    discovery unless this source explicitly points to an issue and the local
    issue-tracker workflow is available.

    ## Standards Source

    For the canonical Standards axis, use the repo's documented standards plus
    the smell baseline from `~/.agents/skills/code-review/SKILL.md`. The
    canonical skill is the single source of truth for that baseline.

    If your environment can dispatch subagents, run the canonical Standards and
    Spec axes in parallel as the skill directs. If it cannot, run the two axes
    yourself in separate passes and keep their findings separated.

    ## Output Format

    Begin directly with the report.

    ### Standards
    Findings from the canonical Standards axis, with file:line evidence and the
    cited documented standard or named baseline smell.

    ### Spec
    Findings from the canonical Spec axis, with file:line evidence and quoted
    requirement text where relevant.

    ### Strengths
    Specific praise for work that is genuinely well-built or notably faithful
    to the plan.

    ### Branch Assessment
    **Ready to merge?** Yes | With fixes | No

    **Reasoning:** One or two sentences. Keep the two axes separate; name the
    worst issue within each axis, matching the canonical skill's aggregation
    rule.
```

**Placeholders:**
- `[MODEL]` — REQUIRED: most capable tier per SKILL.md Model Selection
- `[DESCRIPTION]` — brief summary of what was built
- `[PLAN_OR_REQUIREMENTS]` — what it should do: plan file path, issue directory, task list, or requirements text
- `[BASE_SHA]` — the exact `BASE_COMMIT` recorded from the active branch at invocation
- `[HEAD_SHA]` — ending commit
- `[DIFF_FILE]` — REQUIRED: path from `scripts/review-package BASE_COMMIT HEAD` (prints the unique path it wrote; never enters the controller's context)

**Reviewer returns:** Standards axis, Spec axis, Strengths, and Branch Assessment.
