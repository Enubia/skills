---
name: quality-review
description: Thermonuclear maintainability review for structural simplicity, abstraction quality, and spaghetti growth.
disable-model-invocation: true
---

# Quality Review

An unusually strict review of the requested changes, focused on structure and maintainability. Behavior being correct is the floor, not the bar.

Be **ambitious**. Push past local cleanup. Hunt for **code-judo**: a restructuring that preserves behavior while making the implementation dramatically simpler, smaller, and more direct — one that uses the existing architecture so well the change feels inevitable in hindsight. Prefer deleting complexity over rearranging it.

## The run

1. **Establish scope.** Identify the repo root, current branch, review base, changed files, line counts, and diff range. If the user named a range, use it; otherwise use the merge-base with the default branch. Completion: the report can name the exact branch/range, files reviewed, and `+N / -M`.

2. **Load project context.** Read `CONTEXT.md` and relevant `docs/adr/` entries when present. Completion: findings use the project's vocabulary, and no recommendation contradicts a settled ADR.

3. **Inspect exhaustively.** Read the diff and touched surrounding code until every meaningful change has been checked against the standard. A meaningful change is any touched module boundary, state model, control-flow path, shared helper/API, type contract, or file-size trajectory. Completion: every approval-bar violation has a concrete file reference and fix; every major changed area is either represented by a finding or consciously passed.

4. **Decide the verdict.** Apply the approval bar before writing the report. Completion: the review has one approval call and one highest-leverage fix.

5. **Render the deliverables.** Produce the HTML report, give the inline verdict, and offer to apply the highest-leverage fix. Completion: the user has the absolute report path, the approval call, and the headline blockers.

## The standard

For every meaningful change, push on each of these. They are the review questions, the things to flag, and the remedies — one list, not seven.

- **Code-judo simplification.** Can the change be reframed so whole branches, modes, helpers, or layers disappear? If a refactor moves complexity around without reducing the number of concepts a reader holds in their head, it failed. → Reframe the state model; turn special cases into a simpler default; change the ownership boundary so the feature becomes a natural extension of something that already exists.

- **File size.** A PR pushing a file from under ~1000 lines to over is a strong smell. → Extract helpers, subcomponents, or modules first. Waive only with a compelling reason _and_ a still-clearly-organized result.

- **Spaghetti growth.** New ad-hoc conditionals, scattered special cases, or one-off branches bolted into unrelated flows are a design problem, not a nit. → Push the logic into a dedicated helper, typed model, or dispatcher instead of tangling an existing path. Collapse duplicate branches into one clearer flow.

- **Abstractions earning their keep.** Flag thin wrappers, identity pass-throughs, and "magic" mechanisms that hide simple data-shape assumptions. → Delete the indirection and keep the direct flow. Prefer boring and explicit over clever.

- **Type & boundary cleanliness.** Question unnecessary optionality, `any`, `unknown`, and cast-heavy code that obscures the real invariant; silent fallbacks that paper over unclear contracts. → Make the boundary explicit so the control flow gets simpler.

- **Right layer, canonical helpers.** Flag feature logic leaking into shared paths, implementation details leaking through APIs, and bespoke helpers duplicating something the codebase already has. → Move logic to the package/module that owns the concept; reuse the canonical utility.

- **Orchestration & atomicity.** Independent work serialized for no reason, or related updates that can leave state half-applied. → Parallelize when it also simplifies; restructure into a more atomic flow. Prefer orchestration changes only when they simplify the structure.

## Prioritization

Lead with structural regressions and missed simplifications; then spaghetti/branching; then boundary/type/abstraction problems; then file-size and decomposition. Keep the review to high-conviction maintainability findings — a few strong comments beat a long cosmetic list.

## Approval bar

Approval requires working behavior and structural cleanliness. Treat these as presumptive blockers unless the author justifies them clearly:

- A plausible code-judo move that would delete the preserved complexity went untaken.
- A file crossed ~1000 lines without strong reason.
- Ad-hoc branching made an existing flow more tangled, or feature checks got scattered across shared code.
- An unnecessary wrapper, cast, or optionality muddied the contract.
- A helper was duplicated, or logic landed in the wrong layer when a canonical home exists.

Otherwise leave explicit, actionable feedback pushing for a cleaner decomposition.

## The report

Every run ends by rendering the review as a **single-file HTML report** — the shareable stakeholder deliverable.

Write it to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/quality-review-<timestamp>.html` so each run gets a fresh file. Attempt to open it for the user — `open <path>` on macOS, `xdg-open <path>` on Linux, `start <path>` on Windows — and tell them the absolute path even if the opener fails.

The report uses **Tailwind via CDN** for layout and **Mermaid via CDN** for the diagrams that carry the findings. Lead with a verdict banner (the approval-bar call), then the findings as cards — one per finding, each with a severity badge, category tag, involved files, one-sentence problem and fix, and a before/after diagram where structure is the point (duplication collapsing to one helper, a tangled flow straightened, a leak across a shared path). Mirror the prioritization above in the ordering, and open with a short "done well" callout when the change earned one so the report stays demanding and fair.

See [HTML-REPORT.md](HTML-REPORT.md) for the full scaffold, severity-to-colour mapping, category list, diagram patterns, and styling guidance.

Still give a concise inline verdict in the chat (the headline blockers and the approval call), and end by offering to apply the highest-leverage fix — the report is the artifact, the conversation is where the work continues.

## Tone

Direct, serious, demanding, and professional. Major maintainability issues should read as major issues. If the change makes the codebase messier, say so. If it missed a dramatic simplification, say that too. Examples:

- `this pushes the file past 1k lines — can we decompose first?`
- `another special-case branch in an already busy flow — move it behind its own abstraction?`
- `works, but makes the surrounding code more spaghetti. keep the behavior, restructure the implementation.`
- `feature logic leaking into a shared path — isolate it?`
- `this abstraction isn't earning its keep — keep the direct flow?`
- `why the cast/optional here? make the boundary explicit instead.`
- `bespoke helper for something we already have — reuse the canonical one?`
- `i think there's a code-judo move that makes this much simpler — reframe so these branches disappear?`
