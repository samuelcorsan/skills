# HTML Report Format

The architectural review is rendered as a single self-contained HTML file in the OS temp directory. Put all styling in an inline `<style>` element. Use static HTML and inline SVG for diagrams, including dependency graphs and call flows. The report needs no scripts, fonts, stylesheets, images, or other assets from the network.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Architecture review for {{escaped repo name}}</title>
    <style>
      :root { color-scheme: light; font-family: ui-sans-serif, system-ui, sans-serif; color: #172033; background: #f8f7f4; }
      * { box-sizing: border-box; }
      body { margin: 0; }
      main { max-width: 1040px; margin: auto; padding: 48px 24px 80px; }
      header, .card, #top-recommendation { margin-bottom: 40px; }
      h1, h2, h3, p { margin-top: 0; }
      h1 { font-size: 2rem; letter-spacing: -.04em; }
      h2 { font-size: 1.4rem; letter-spacing: -.02em; }
      h3 { font-size: .85rem; text-transform: uppercase; letter-spacing: .08em; }
      .meta, .legend, .files { color: #475569; font-size: .9rem; }
      .legend { display: flex; gap: 16px; flex-wrap: wrap; }
      .card, #top-recommendation { padding: 28px; border: 1px solid #dbe1e7; border-radius: 16px; background: white; }
      .badges { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 16px; }
      .badge { padding: 4px 9px; border-radius: 999px; background: #e2e8f0; color: #334155; font-size: .75rem; font-weight: 700; }
      .strong { background: #d1fae5; color: #065f46; }
      .exploring { background: #fef3c7; color: #92400e; }
      .diagrams { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 16px; margin: 24px 0; }
      .diagram { min-width: 0; padding: 16px; border: 1px solid #dbe1e7; border-radius: 12px; background: #fcfcfb; }
      .diagram svg { display: block; width: 100%; height: auto; max-height: 320px; }
      .node { fill: #fff; stroke: #64748b; stroke-width: 2; }
      .deep-node { fill: #172033; stroke: #172033; stroke-width: 2; }
      .seam { fill: none; stroke: #64748b; stroke-width: 2; stroke-dasharray: 5 5; }
      .arrow { fill: none; stroke: #475569; stroke-width: 2; }
      .leak { fill: none; stroke: #dc2626; stroke-width: 2; }
      .node-label { fill: #172033; font: 13px ui-sans-serif, system-ui, sans-serif; }
      .deep-label { fill: #fff; font: 13px ui-sans-serif, system-ui, sans-serif; }
      .details { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 16px; }
      .files { font-family: ui-monospace, monospace; overflow-wrap: anywhere; }
      .warning { padding: 12px; border-radius: 8px; background: #fffbeb; color: #92400e; }
      a { color: #065f46; }
      @media (max-width: 720px) { main { padding: 24px 16px; } .diagrams, .details { grid-template-columns: 1fr; } }
    </style>
  </head>
  <body>
    <main>
      <header>
        <h1>Architecture review for {{escaped repo name}}</h1>
        <p class="meta">{{escaped date}}</p>
        <div class="legend">Solid box = module · Dashed line = seam · Red arrow = leakage · Dark box = deep module</div>
      </header>
      <section id="candidates" aria-label="Candidates">
        <!-- Repeat one candidate article per opportunity, using the card structure below. -->
      </section>
      <section id="top-recommendation" aria-labelledby="top-heading">
        <h2 id="top-heading">Top recommendation</h2>
        <p><a href="#candidate-1">{{escaped candidate title}}</a>: {{escaped reason}}</p>
      </section>
    </main>
  </body>
</html>
```

Replace placeholders with escaped text. For repository-derived text in HTML or SVG text nodes, encode `&` as `&amp;`, `<` as `&lt;`, and `>` as `&gt;`. Encode quotes as `&quot;` and `&#39;` when text appears in an attribute. Use fixed, locally generated IDs such as `candidate-1` for anchors rather than repository-derived URL values. Keep repository-derived content out of raw HTML, CSS, URL, SVG markup, event attributes, and external asset references. Build the diagram shapes yourself and place escaped labels in text nodes. These rules apply to repo names, paths, candidate titles, glossary terms, and ADR excerpts alike.

## Header

Repo name, date, and a compact legend: solid box = module, dashed line = seam, red arrow = leakage, thick dark box = deep module. No introduction paragraph. Straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the glossary terms (from the `/codebase-design` skill) without ceremony.

Each candidate is one `<article class="card" id="candidate-1">`, with a distinct locally generated ID for each card:

- **Title**: short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Badge row**: recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the dependency category (`in-process`, `local-substitutable`, `ports & adapters`, `mock`). Use `.badge` with `.strong` or `.exploring` when applicable.
- **Files**: monospaced list in `.files`.
- **Before / After diagram**: the centrepiece. Two `.diagram` columns inside `.diagrams`. See patterns below.
- **Problem**: one sentence. What hurts.
- **Solution**: one sentence. What changes.
- **Wins**: bullets, ≤6 words each. e.g. "Tests hit one interface", "Pricing logic stops leaking", "Delete 4 shallow wrappers".
- **ADR callout** (if applicable): one line in `.warning`.

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same. Variety is part of the point. Give each diagram an accessible name with its heading and, for SVG, a `<title>` with a distinct locally generated ID.

### Static dependency graph or call flow

Use inline SVG nodes, connectors, and `<text>` labels when the point is "X calls Y calls Z, and look at the mess." Lay out nodes on a fixed `viewBox` with enough room for long labels; use the `.node`, `.deep-node`, `.arrow`, `.leak`, and `.seam` styles. Use a red connector only for an actual leak across a seam. For a sequence, use vertical lifelines and numbered, labelled arrows. Every SVG label is escaped text.

```html
<div class="diagram">
  <h3>Before</h3>
  <svg viewBox="0 0 440 180" role="img" aria-labelledby="flow-title">
    <title id="flow-title">Order intake crosses a pricing seam</title>
    <rect class="node" x="12" y="48" width="120" height="56" rx="8" />
    <text class="node-label" x="24" y="81">Order intake</text>
    <path class="arrow" d="M132 76 H206" />
    <rect class="node" x="206" y="48" width="100" height="56" rx="8" />
    <text class="node-label" x="218" y="81">Order store</text>
    <path class="seam" d="M324 24 V132" />
    <path class="leak" d="M306 76 H344" />
    <rect class="node" x="344" y="48" width="84" height="56" rx="8" />
    <text class="node-label" x="354" y="81">Pricing</text>
  </svg>
</div>
```

For repo-specific labels, replace the example text using the escaping rules above. Keep shapes and label placement visible at narrow widths. Put long labels on multiple `<text>` lines or use a wider viewBox, rather than clipping them.

### Hand-built boxes and arrows

Modules as bordered `<div>` elements with labels. Arrows as inline SVG `<line>` or `<path>` elements in a relative container. Reach for this when the "after" diagram should feel like one thick-bordered deep module with greyed-out internals.

### Cross-section (good for layered shallowness)

Stack horizontal bands to show modules a call passes through. Before: six thin bands each doing little. After: one thick band labelled with the consolidated responsibility.

### Mass diagram (good for "interface as wide as implementation")

Two rectangles per module: one for interface surface area, one for implementation. Before: interface rectangle is nearly as tall as the implementation rectangle (shallow). After: interface rectangle is short, implementation rectangle is tall (deep).

### Call-graph collapse

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Serif optional for headings, using a local system font.
- Colour sparingly: one accent (emerald or indigo) plus red for leakage and amber for warnings.
- Keep diagrams ~320px tall so before/after sits comfortably side by side without scrolling.
- Use small uppercase labels inside diagrams so they read as schematic, not as UI.
- Keep the report static. Do not add scripts, external imports, event handlers, or network assets for styling or diagrams.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain English, concise, but the architectural nouns and verbs come straight from the `/codebase-design` skill. Concision is not an excuse to drift.

**Use exactly:** module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality.

**Never substitute:** component, service, unit (for module) · API, signature (for interface) · boundary (for seam) · layer, wrapper (for module, when you mean module).

**Phrasings that fit the style:**

- "Order intake module is shallow: interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in prod, in-memory in tests."

**Wins bullets** name the gain in glossary terms: *"locality: bugs concentrate in one module"*, *"leverage: one interface, N call sites"*, *"interface shrinks; implementation absorbs the wrappers"*. Don't write *"easier to maintain"* or *"cleaner code"*, because those terms aren't in the glossary and don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in the `/codebase-design` glossary, reach for one that is before inventing a new one.
