# HTML Report Format

The quality review is rendered as a single-file HTML report in the OS temp directory. Tailwind and Mermaid both come from CDNs, so the file is shareable as one artifact but network-dependent for full styling and diagrams. Mermaid handles graph-shaped diagrams (duplication fan-out, call flow, decision branches) reliably; hand-built divs and inline code blocks handle the more editorial visuals. Mix the two so each finding gets the visual form that carries it best.

The diagrams carry the findings. Prose is sparse: a one-sentence problem and a one-sentence fix per card. If a finding needs a paragraph to land, draw it instead.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Quality review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose",
        flowchart: { htmlLabels: true, curve: "basis" } });
    </script>
    <style>
      .mermaid { display: flex; justify-content: center; }
      .card-anchor { scroll-margin-top: 2rem; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="done-well">...</section>
      <section id="findings" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, and the review scope (branch or commit range). Then:

- **Stat chips** — a small grid: files reviewed, lines (`+N / −M`), blocker count, total findings.
- **Verdict banner** — the approval-bar call, up top, so the bottom line reads first. Red left-border (`border-l-4 border-rose-500 bg-rose-50`) when not approvable; emerald when approvable. One sentence naming the gating issues.
- **Legend** — compact: blocker (rose), major (amber), minor (slate), done well (emerald), plus the category list.

Start directly with the verdict, then the findings.

## Done-well callout

When the change earned praise — a real code-judo simplification, complexity deleted rather than moved — open with one emerald-tinted card before the findings. Include the before/after diagram of the good move. Keep it honest: reserve the callout for earned praise. The report should be demanding and fair.

## Finding card

Each finding is one `<article class="card-anchor rounded-xl border border-slate-200 bg-white p-6 space-y-5">` with an `id` for anchor links.

- **Title** — numbered, short, names the problem (e.g. "Back-navigation block copy-pasted 11 times").
- **Badge row** — severity badge plus category tag.
- **Files** — monospaced list, `font-mono text-sm text-slate-500`.
- **Diagram** — the centrepiece for structural findings. Before/after in two columns. See patterns below. Boundary/type findings can show a dark code block instead of a diagram.
- **Problem / Fix** — a two-column `<dl>`, one sentence each. `Problem`: what hurts. `Fix`: what changes.
- **Impact bullets** (optional) — ≤6 words each, naming the gain: "one interface, 11 call sites", "back-policy change lands in one place". Skip "easier to maintain" / "cleaner code".

Lead with structural regressions and missed simplifications, then spaghetti/branching, then boundary/type/abstraction, then file-size/decomposition. Keep the report to high-conviction cards. Roll all minors into a single trailing card.

## Severity and category

Severity answers whether the finding blocks approval; category answers what kind of maintainability issue it is.

### Severity

- `Blocker` — rose (`bg-rose-100 text-rose-700`). Any approval-bar violation: structural regression, missed code-judo, tangled shared flow, unjustified 1k+ file crossing, duplicated canonical helper, or muddied contract that changes how readers must reason about the code.
- `Major` — amber (`bg-amber-100 text-amber-700`). Important maintainability issue that should be fixed, but does not by itself block approval.
- `Minor` — slate (`bg-slate-100 text-slate-600`). Cosmetic or low-risk cleanup; cluster these into one card.

### Category

Use one category tag per finding:

- `Code-judo`
- `File size`
- `Spaghetti`
- `Abstraction`
- `Boundary/type`
- `Layer/canonical helper`
- `Orchestration/atomicity`

Examples: `Blocker · Spaghetti`, `Blocker · Code-judo`, `Major · Boundary/type`, `Minor · Abstraction`.

## Diagram patterns

Pick the pattern that fits the finding. Mix them so adjacent cards have distinct shapes.

### Duplication fan-out

Use for copy-paste or a missing helper. Before: N call sites each pointing at their own identical block, all tinted red. After: the same N pointing at one shared helper, tinted emerald.

```html
<div class="rounded-lg border border-slate-200 p-4">
  <div class="text-xs uppercase tracking-wider text-slate-400 mb-2">Before — N copies</div>
  <pre class="mermaid">
    flowchart TB
      H1[useFoo] --> D1["canGoBack ? back : replace(fallback)"]
      H2[useBar] --> D2["canGoBack ? back : replace(fallback)"]
      H3["…more"] --> D3["canGoBack ? back : replace(fallback)"]
      classDef dup fill:#fee2e2,stroke:#dc2626,color:#000;
      class D1,D2,D3 dup
  </pre>
</div>
```

### Leak across a shared path

Use for feature logic or placeholders reachable in a live flow. `flowchart LR` with the offending edge/target tinted red.

```
flowchart LR
  A[AnalysisPending] -- "status = Decision" --> B["DecisionScreen<br/>TODO"]
  classDef leak fill:#fee2e2,stroke:#dc2626,stroke-width:2px,color:#000;
  class B leak
```

### Decision branch

Use for special-cases bolted into a shared handler. A `flowchart TB` with a `{condition}` node and the special-case branch tinted amber shows the one-off `if` ahead of the generic path.

### Before/after collapse

Use for code-judo wins, especially in the done-well card. Before: a stateful tangle (hook with `useEffect` + `useState` orchestration) as separate nodes. After: the same behaviour collapsed into one node/subgraph, tinted emerald.

### Dark code block

Use for boundary/type findings. When the issue is a cast or muddied contract, a `<pre>` with `bg-slate-900 text-slate-200 text-xs font-mono` showing the offending snippet reads better than a diagram. Use a graph only when structure is the point.

## Top recommendation section

One larger card, dark gradient (`background: linear-gradient(135deg, #0f172a, #1e293b)`). The single highest-leverage fix — usually the cheapest blocker to clear or the biggest code-judo win — one sentence on why, and an anchor link to its finding card. Close with a one-line footer crediting `/quality-review` and naming the reviewed branch/commit.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. `font-serif` for headings pairs well with stone/slate.
- Colour sparingly: rose for blockers, amber for major findings, emerald for done-well and the recommendation, slate for everything else.
- Keep diagrams ~320px tall so before/after sits side by side without scrolling.
- Use `text-xs uppercase tracking-wider` for labels inside diagrams — they should read as schematic, not as UI.
- The only scripts are the Tailwind CDN and the Mermaid ESM import. Otherwise static — no app code, no interactivity beyond Mermaid's own rendering and anchor links.
- Mermaid node labels: wrap multi-line text in quotes and use `<br/>`; escape `&` as `&amp;` in HTML-level labels.

## Tone in the report

Same voice as the review: direct, serious, demanding, and professional. No hedging, no throat-clearing. If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. The verdict banner says plainly whether it's approvable; the cards say plainly what's wrong and what to do.
