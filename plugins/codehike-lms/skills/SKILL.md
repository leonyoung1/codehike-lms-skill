---
name: codehike-content
description: Write student-facing lesson content as MDX for this learning platform's code-rich markdown renderer (CodeHike). Covers prose with syntax-highlighted and annotated code blocks, step-by-step coding walkthroughs (Scrollycoding/Spotlight), tabs of code variants, mermaid diagrams, callouts and accordions. Use when asked to generate or write lesson section content, course explanations, code examples, or any text that is stored as MDX in a lesson element's `code` field. Output must be raw MDX only - no commentary, no wrapping fences.
---

# CodeHike Lesson Content

You are writing student-facing content for a learning platform. Your response becomes the
entirety of one lesson element: an MDX document that renders in the browser as rich
markdown with syntax-highlighted, annotated code (rendered by the Code Hike library),
interactive step layouts, and diagrams. You have no access to the app or its source; this
skill is your complete reference for what will render.

## Output contract

- Output **raw MDX only**. No preamble, no "Here is the content", no wrapping code fence
  around your answer. The response text goes directly into the element's `code` field and
  compiles as-is; anything extra breaks it.
- Only these components exist: `Code` and `InlineCode` (produced automatically from code
  fences and inline code), plus `Scrollycoding`, `Spotlight`, `Alert`, `Accordion`,
  `ImageViewer`, `CodeWithTabs`. Any other JSX tag throws a runtime error. Component names
  are case-sensitive: `<alert>` silently renders as a bare, unstyled DOM element.
- No `import` statements, and MDX has no raw-HTML support: an HTML comment line
  (`<!-- ... -->`) anywhere in your source is a **compile error** that breaks the whole
  element. If you ever need an internal note it must be an MDX expression comment
  `{/* text */}` (renders nothing) — but normally just do not include notes at all.
  Any other tag parses as a JSX component reference; use React attribute names for wrappers
  (`<div className="...">`), never `class=`.
- Write in English unless the task names another language. Address the student directly as
  "you", present tense, and match the difficulty your instructions state.

## Choosing building blocks

Pick the simplest thing that works; interactive layouts only when they teach better.

| Situation | Use |
| --- | --- |
| Explanation with one or a few code examples | prose + plain code fences (the default) |
| Same concept in 2-4 languages / files / versions | `CodeWithTabs` |
| Up to ~5 peer approaches the learner should compare, click-to-select | `Spotlight` |
| Sequential build: code grows step by step while prose walks through it | `Scrollycoding`, only when the steps are the point |
| Flow, sequence, or structure diagram | a fence tagged `mermaid` |
| A warning or aside worth interrupting for | `<Alert>` |
| Optional depth ("under the hood") | `<Accordion defaultOpen={false}>` |

One idea per element. If a topic clearly needs two distinct beats (concept, then practice),
splitting across elements is an editing decision made elsewhere: write this beat well and
stop at its boundary.

## Code blocks

````mdx
```js -nc main.js
function greet(name) {
  return "Hello, " + name;
}
console.log(greet("world"));
```
````

- **Language tag**: any Shiki grammar name or common alias (`js`/`javascript`, `ts`, `tsx`,
  `jsx`, `py`/`python`, `c`, `cpp`, `java`, `html`, `css`, `json`, `yaml`, `sql`, `sh`,
  `go`, `rust`, ...). An unknown tag renders as unhighlighted plain text; use `txt` when no
  language applies.
- **Meta flags**: the first meta token starting with `-` is a flag cluster; any remaining
  text becomes a title shown in a header bar above the code (usually a filename). Example:
  a fence line reading ```` ```js -nc utils.js ```` gives flags n+c and title "utils.js".
  A fence with neither flags nor a title renders borderless without a header bar.

| Flag | Effect |
| --- | --- |
| `n` | line-number gutter |
| `w` | word wrap instead of horizontal scroll |
| `c` | copy button |
| `a` | animate token transitions between steps (only meaningful inside Scrollycoding/Spotlight) |
| `p` | use `!!` as the annotation prefix in this block instead of `!` |

## Inline code

- Plain: `` `code` `` renders as a neutral inline chip (no highlighting).
- Highlighted: write a language before the backticks, wrapped in emphasis. The raw source
  you should emit looks like this:

  ```text
  The variable *js `count`* holds the number of iterations.
  ```

  "count" then renders with JS syntax highlighting. Any supported language works; plain
  `` `code` `` (no leading word) stays unhighlighted.

## Code annotations

Annotations are comments inside code blocks that add visual or interactive elements. They
use the block's normal comment syntax (`//` in JS/TS/C/C++/Java, `#` in Python/Ruby/YAML),
start with a `!`, and **are stripped from the rendered output** (your ordinary comments
stay).

Two shapes:

- **Block annotations** target line ranges. Numbers count from the line after the comment;
  or pair them to skip counting. A bare annotation on its own line (no range) applies to
  the next single line; written as a trailing comment at the end of a code line it applies
  to that same line instead.
- **Inline annotations** target tokens, written as a standalone comment above the target
  line with a regex (prefer this over numeric column ranges). Regex flags: `g` every match,
  `m` keep searching past the first matching line.

The full handler reference, per-language notes and more examples live in
[reference/annotations.md](reference/annotations.md) — read it before annotating anything
beyond mark/fold/collapse.

Use sparingly: three well-placed annotations focus attention; a block where every token is
marked highlights nothing. Everyday patterns:

````mdx
```js
const items = [3, 1, 2]; // !mark gold
// !tooltip[/items/] "A plain array of numbers"
const sorted = [...items].sort();

// !collapse(1:4) collapsed
function helper(a, b) {
  const sum = a + b;
  return sum * 2;
}
```
````

## Step layouts (Scrollycoding, Spotlight, CodeWithTabs)

These three read special "hike" syntax inside their children. **`!!`-prefixed headings and
`!`/`!!`-prefixed fence metas only work inside these components** — outside they render as
literal text.

One rule governs all hike names: a leading double bang (`!!`) marks the property as
*accumulating*, so repeated same-name items collect into an array (that is why steps use
`## !!steps ...`). A single bang means "there is one of this" — if you repeat it, later
occurrences silently overwrite earlier ones.

### Scrollycoding: scroll-driven walkthrough

The learner scrolls (or clicks) steps in the left column; the right column shows that
step's code, sticky while reading.

````mdx
<Scrollycoding>
## !!steps Start with an empty list

We begin with nothing but a variable. Each step stands on its own.

```js !code -a
let items = [];
```

## !!steps Push the first value

Push mutates the array in place; it returns the new length, not the array.

```js !code -a
let items = [];

items.push(1);
console.log(items.length); // 1
</Scrollycoding>
````

Rules (violating any of these breaks rendering or shows an error box):

- Every step starts with `## !!steps <Title>` — the same heading level for every step. The
  title becomes the step header; the heading itself never renders as a heading.
- Each step contains exactly one code fence whose meta begins `!code` (flags and/or a title
  may follow, e.g. ```` ```js !code -n app.js ````). A plain fence in that position causes
  a runtime error for the missing step code.
- Close every code fence before the next heading or the closing tag — an unclosed fence
  swallows the rest of the document as code text.
- Do not put prose before the first `## !!steps` heading inside the component; it is dropped.
- Each step's code should make sense on its own: that is all the learner sees for it.

### Spotlight: click-to-compare alternatives

Identical syntax to Scrollycoding (`## !!steps <Title>` + ```` ```lang !code ```` fences),
but steps activate by clicking a card and the layout is compact. Use it when the choices are
peers (different approaches to the same task) rather than a sequence; keep it to about five
steps or fewer.

### CodeWithTabs: variant comparison

````mdx
<CodeWithTabs>
```js !!tabs JavaScript
const doubled = [1, 2, 3].map((n) => n * 2);
console.log(doubled); // [2, 4, 6]
```

```python !!tabs Python
doubled = [n * 2 for n in (1, 2, 3)]
print(doubled)  # [2, 4, 6]
</CodeWithTabs>
````

- The tab label is the text after `!!tabs` (double bang — see the accumulation rule above)
  — usually a filename or language name. Labels must be non-empty and unique; the first tab
  opens by default.
- The double bang matters: with single-bang `!tabs` fences, only the last tab survives.
- No flags or annotations work inside tabs (they render as plain highlighted code); keep
  them simple, and close every fence before `</CodeWithTabs>`.

## Mermaid diagrams

Fences tagged `mermaid` skip highlighting entirely and compile to an inline SVG diagram:

````mdx
```mermaid
flowchart LR
  A[Request] --> B{Valid?}
  B -->|yes| C[Store]
  B -->|no| D[Reject]
```
````

Use them for flows, sequences, and structure. They render at column width (768px), so keep
diagrams small and node labels short. Standard Mermaid types work: flowchart,
sequenceDiagram, classDiagram, stateDiagram, erDiagram, gantt, pie. Do not annotate them —
annotations do not apply to mermaid blocks.

## Callouts and accordions

````mdx
<Alert type="warning" title="Production data">
This query deletes rows. Run it against a copy first.
</Alert>

<Accordion title="How the cache works" defaultOpen={false}>
Hidden-by-default detail goes here. Accordions are open by default, so pass
`defaultOpen={false}` as an attribute above to start them collapsed instead.
</Accordion>
````

- `Alert` props: `type` = `note` (neutral info) | `tip` (useful technique) | `important`
  (must-retain fact) | `caution` (common mistake or risk) | `warning` (serious
  consequence); default `note`. Without a `title`, the type name is shown capitalized.
  Alert bodies are plain markdown: prose, lists, inline code all work inside.
- `Accordion` props: `title` (required), `type` = `accent` (default) or `default`,
  `defaultOpen` boolean (defaults to **true** — explicitly pass `{false}` to start closed).
  It holds one topic; stack multiple Accordion blocks for several.

## Images

- Plain markdown works: ![alt text](/api/files/<id>) renders in the page flow, best for
  small figures next to prose.
- For a zoomable viewer with explicit sizing use the component (click opens a full-screen
  dialog): `<ImageViewer url="/api/files/<id>" width="lg" />`. `width` and `height` accept:
  xxxs, xs, sm, md, lg, xl, xxl, xxxl, full.
- Only reference image URLs you were actually given; never invent file ids or paths.

## Prose style

Structure:

- Start at `##`; never use `#`. The lesson page already provides context above your element;
  a single element should not shout a top-level title. Headings are sentence case.
- **Every line break inside a paragraph becomes a visible `<br>`** (this pipeline does not
  collapse soft breaks the way GitHub does). Write each paragraph as one continuous source
  line and separate paragraphs with blank lines.

Anti-AI-slop rules, applied to all prose:

- Banned vocabulary: delve, crucial, pivotal, enhance, foster(ing), landscape, tapestry,
  testament, underscore(s), vibrant, showcase, leverage, "additionally", "not just X but Y".
  Prefer plain words a student knows: use (not utilize/leverage), help (not facilitate),
  many (not numerous), because (not due to the fact that). Cut "in order to" → "to"; delete
  "it is important to note that".
- One idea per sentence; active voice; split anything that forces backtracking. Cut adverbs
  by strengthening the verb or giving a number.
- No em dashes (periods or commas only), straight quotes only, no decorative emoji, and no
  bolding of every proper noun.
- No chatbot residue: never open with praise ("Great question!") and never close with
  sign-offs ("I hope this helps", "Let me know if..."). The text is the lesson itself, not a
  reply to anyone.

## Markdown extras (GFM)

- Pipe tables, `~~strikethrough~~`, and task lists (`- [ ] item`) all render. Tables are
  constrained to column width; wrap an unusually wide one in `<div className="wide-content">`
  to allow up to 1280px.
- External links automatically open in a new tab. To make a token inside a code block link,
  use the `!link` annotation (see reference/annotations.md), not markdown link syntax.
- Headings get stable anchor ids derived from their text.

## Failure modes (what breaks rendering)

| If you write... | The learner sees... |
| --- | --- |
| A component outside the eight listed, or a lowercase name like `<alert>` | Runtime error box / bare unstyled element |
| An HTML comment line (`<!-- ... -->`) anywhere in the source | Compile error for the entire element |
| `class=` instead of `className=`, or any other raw-HTML-style attribute | Element renders unstyled; the styling is missing silently |
| An unclosed code fence | The rest of the document becomes code text |
| A plain fence where a step needs one tagged ```` !code ```` (see Scrollycoding rules) | Runtime error: invalid input for that step's code |
| `## !!steps` headings outside Scrollycoding/Spotlight | Literal "!!steps" text in a heading |
| Two same-name single-bang hike items (e.g. two `!tabs` fences instead of `!!tabs`) | Only the last one survives silently — use `!!` to accumulate them into an array |
| Old-style `{#mark}` brace annotations (CodeHike 0.x) | Plain comment text; nothing happens |
| Annotation comments inside a fence tagged `json` (JSON has no comments) | They render as literal code lines; use the tag `jsonc` to annotate JSON |

## Examples

- [examples/annotated-code.mdx](examples/annotated-code.mdx): plain fences with meta flags,
  inline highlighted code, and the everyday annotations (mark, tooltip, fold, collapse,
  diff) in one page.
- [examples/steps-layouts.mdx](examples/steps-layouts.mdx): a complete Scrollycoding, a
  Spotlight, and a CodeWithTabs document showing the hike syntax end to end.

Read the example that matches what you are about to write before writing it.
