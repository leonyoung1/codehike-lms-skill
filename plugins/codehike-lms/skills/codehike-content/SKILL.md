---
name: codehike-content
description: "Write student-facing lesson content as raw CodeHike MDX for an element's content field — the platform's code-rich markdown with syntax-highlighted and annotated code blocks, step-by-step coding walkthroughs (Scrollycoding/Spotlight), tabs of code variants, mermaid diagrams, callouts and accordions. Use when asked to produce or write that MDX: a full lesson section for a markdown element's `code` field, or the content of any other CodeHike-routed field (instructional notes, multiple-choice question/hint, free-response hint/answer, file-submission prompt). Output must be raw MDX only - no commentary, no wrapping fences. Not for: element records `{type, data}` (lms-elements), playground file sets — SandpackFiles maps or flat-root zips (lms-playground), whole backup JSON files (lms-backup)."
---

# CodeHike Lesson Content

You are writing student-facing content for a learning platform. Your response becomes one
element's MDX content field — most often the entirety of a markdown lesson section: an MDX
document that renders in the browser as rich markdown with syntax-highlighted, annotated code
(rendered by the Code Hike library), interactive step layouts, and diagrams. You have no access
to the app or its source; this skill is your complete reference for what will render.

## Where your output goes

Your raw MDX lands **verbatim** in an element's content fields — you emit plain text only:

- A markdown element's `data.code` — the section-level lesson content this skill was named
  for.
- Instructional note fields (the per-element staff notes) — same pipeline, but the audience is
  the instructor, not the student: write facilitation tips, coaching cues, and answers for the
  person teaching; the NoteBlock scaffolds below exist for exactly that.
- `multiple-choice.data.question` and the free-response / file-submission question fields —
  short/inline prose rendered inside the component (no heading root).
- **lms-backup** embeds your output verbatim into backup JSON files; JSON escaping is its job,
  not yours.
- **lms-elements** delegates every CodeHike-routed field to you and places your output verbatim
  in those fields of the record it authors.

The full field→parser routing table (which field of which type compiles through this pipeline vs
plain GFM vs raw text) lives in **lms-elements**.

## Output contract

- Output **raw MDX only**. No preamble, no "Here is the content", no wrapping code fence
  around your answer. The response text goes directly into the element's `code` field and
  compiles as-is; anything extra breaks it.
- Only these components exist: `Code` and `InlineCode` (produced automatically from code
  fences and inline code), plus `Scrollycoding`, `Spotlight`, `Alert`, `Accordion`,
  `NoteBlock`, `Subsection`, `ImageViewer`, `CodeWithTabs`, `CodeMentions`, and `Tooltip`. Any other JSX
  tag throws a runtime error *(lowercase intrinsic elements such as `<div>` and `<iframe>` are HTML, not
  components: they render normally; see Embeds)*. Component names are case-sensitive:
  `<alert>` silently renders as a bare, unstyled DOM element.
- No `import` statements, and MDX has no raw-HTML support: an HTML comment line
  (`<!-- ... -->`) anywhere in your source is a **compile error** that breaks the whole
  element. If you ever need an internal note it must be an MDX expression comment
  `{/* text */}` (renders nothing) — but normally just do not include notes at all.
  Any other tag parses as a JSX component reference; use React attribute names for wrappers
  (`<div className="...">`), never `class=`.
- Write in English unless the task names another language. Address the student directly as
  "you", present tense, and match the difficulty your instructions state; when they name no
  audience, default to a fourteen-year-old beginner in the topic being taught: assume
  school-level reading ability, but nothing about the subject itself.

## Choosing building blocks

Pick the simplest thing that works; interactive layouts only when they teach better.

| Situation | Use |
| --- | --- |
| Explanation with one or a few code examples | prose + plain code fences (the default) |
| Same concept in 2-4 languages / files / versions | `CodeWithTabs` |
| Prose that should light up specific lines of a code block on hover or focus | `CodeMentions` (code mentions) |
| Up to ~5 peer approaches the learner should compare, click-to-select | `Spotlight` |
| Sequential build: code grows step by step while prose walks through it | `Scrollycoding`, only when the steps are the point |
| Flow, sequence, or structure diagram | a fence tagged `mermaid` |
| A warning or aside worth interrupting for | `<Alert>` |
| A word or phrase worth defining on hover | `<Tooltip tip="…">` |
| Optional depth ("under the hood") | `<Accordion defaultOpen={false}>` |
| Structured staff-note block (tips / check-for-understanding / answer) | `<NoteBlock type="…">` |
| A named activity or Do Now beat inside a section | `<Subsection type="…" title="…"/>` |

## Designing for the learner

- Introduce terms and data structures before they are needed to follow what comes next.
- Explain why something exists: problem, operation, consequence. Prefer "this guard prevents
  an empty request" over "here we add an if statement".
- Open with the concrete goal: what the student will build or be able to explain by the end
  of this beat, and what success looks like.
- Titles name the idea or action ("Stop when the queue is empty"), not position in a sequence
  ("Step 3"). This holds for step headings, Alert titles, and Accordion titles alike.
- One coherent example throughout: the same scenario and data from start to finish, minimal
  meaningful names, long expressions broken across lines. When it spans multiple files,
  title each fence and say how the pieces relate.
- Connect claims to concrete outcomes: one specific input (or situation) with its expected
  result, and which lines (or passages) produce it. For anything that changes over time
  (loops, recursion, async flows, multi-step processes), show the changing state between
  beats in prose; use `pill`, `ruler`, or a mermaid diagram when a picture says it better.
- When a mistake (or misreading) teaches, teach it: show the wrong version, name its symptom,
  then repair it (two adjacent steps for a scrolly walkthrough, `!diff` inside one fence).
  Point students at the memorable ones with a `caution` Alert.
- Close with a small transfer exercise: adapt the example to a related case, state the
  desired behavior, and hint back at the concept just taught. A task-list item renders as a
  checkbox, which fits this well.

## Code blocks

````mdx
```js main.js -nc
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
- **Meta flags**: after the language tag, write the title first (usually a filename) and
  then an optional flag cluster, which is one `-` token such as `-nc`. Example: ```` ```js utils.js -nc ````
  gives flags n+c and title "utils.js". Never put the flag before the title: `js -nc utils.js`
  keeps the "-nc" inside the title, so the header bar shows "-nc utils.js" verbatim.
  A fence with neither flags nor a title renders borderless without a header bar.
- **No backtick on the fence line itself**: CommonMark forbids them inside an info string,
  so any fence whose language tag, flags, title, or label contains one (the typical
  trigger is highlighted inline syntax in a tab label) is not parsed as a code fence at
  all. The text then renders as prose where the syntax in it activates for real, and the
  next ``` opens a junk block that swallows everything until the next closing fence.

| Flag | Effect |
| --- | --- |
| `n` | line-number gutter |
| `w` | word wrap instead of horizontal scroll |
| `c` | copy button |
| `a` | animate token transitions between steps (only meaningful inside Scrollycoding/Spotlight) |
| `p` | use `!!` as the annotation prefix in this block instead of `!` |

## Inline code

Default to **highlighted** inline code for real tokens of code (variables, functions,
keywords, method calls): write a language before the backticks, wrapped in emphasis. The
raw source you should emit looks like this:

  ```text
  The variable *js `count`* holds the number of iterations.
  ```

"count" then renders as a chip with JS syntax highlighting. Any supported language works;
always include the language word, because an emphasized code without one (like `` *`count`* ``)
silently highlights as JSX instead of whatever you had in mind.

Plain: `` `code` `` renders as a neutral inline chip (no highlighting). Reserve it for
things that are not code in any single language: file paths (`src/lib/store.ts`), CLI
flags (`--no-pager`), and bare construct names where the current language does not apply.

Where it works, anywhere prose works: paragraphs, lists, blockquotes, link text, headings
(the span contributes nothing to the anchor id, so a double hyphen marks its place in the
slug), table cells, Alert and Accordion bodies, and Scrollycoding/Spotlight step bodies.
It never works inside JSX attribute values or on fence meta lines; see Failure modes for
what those render as instead.

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
- Step titles are plain text only. Highlighted inline code in one does not render, and
  everything from it on is dropped silently (a truncated title, or an empty one if the span
  comes first) with no error to catch it.
- Each step contains exactly one code fence whose meta begins `!code` (flags and/or a title
  may follow, e.g. ```` ```js !code app.js -n ````). A plain fence in that position causes
  a runtime error for the missing step code.
- Close every code fence before the next heading or the closing tag — an unclosed fence
  swallows the rest of the document as code text.
- Do not put prose before the first `## !!steps` heading inside the component; it is dropped.
- Each step's code should make sense on its own: that is all the learner sees for it.

Design rules (these do not break rendering, they decide whether the walkthrough teaches):

- Each step's code shows what its prose explains, plus enough surrounding context to make
  sense (signatures, conditions). If a step revisits an earlier line of a long block that
  stays up across several steps, repeat the same code and only move `!focus` or `mark`. Mark
  at most one important line inside the focused region.
- Changes between steps are meaningful: keep unchanged lines stable (the `-a` diff relies on
  it) and say what changed and why when the code differs from the previous step. Explaining
  an earlier line is not a reason to rewrite it; move the focus instead.

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
  opens by default. They are plain text taken from the fence line, so highlighted inline
  syntax in one renders literally as asterisks and backticks.
- The double bang matters: with single-bang `!tabs` fences, only the last tab survives.
- No flags or annotations work inside tabs (they render as plain highlighted code); keep
  them simple, and close every fence before `</CodeWithTabs>`.

## Code mentions (hover/focus highlight)

Wrap prose and one or more code fences in `<CodeMentions>`. Ordinary markdown links whose href
starts with `hover:` inside that wrapper render as mention chips; while a chip is hovered or
keyboard-focused, every line of the group's tagged fences that does not carry its id drops to
low opacity while matching lines stay fully lit. Clicking a chip does nothing — it is not a
link anywhere.

````mdx
<CodeMentions>
The [base case](hover:one) returns 1 directly; the [recursive case](hover:two) multiplies by the result of calling itself again.

```js
function factorial(n) {
  if (n === 0) {
    // !mention one
    return 1;
  } else {
    // !mention two
    return n * factorial(n - 1);
  }
}
```
</CodeMentions>
````

Rules (violating any of these leaves mentions inert or highlights the wrong lines):

- The id after `!mention` must match the text after `hover:` exactly: case-sensitive, and a
  single id contains no spaces. Tag on its own line directly above the target, or as a
  trailing comment (`return 1; // !mention one`) to tag that line itself; standard block
  ranges also work (`// !mention(2:4) two`).
- Every fence you want dimmed must contain at least one `!mention` tag — untagged fences are
  not part of the group and stay fully lit.
- Mention links only work inside a `<CodeMentions>` wrapper; outside, they render as styled
  chips that highlight nothing because nothing listens for them. Do not nest wrappers.
- The annotation comments strip from the rendered code like every other annotation comment,
  so the block stays clean for students who read it straight through.

Use it to point at where a symbol does its work ("see how `fetchJson` gets called") without
scroll-drift between prose and block. Keep it to about two to four mentions per group; beyond
that, plain prose with line references teaches better.

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
annotations do not apply to mermaid blocks, and label text inside nodes renders exactly as
written (no markdown, no highlighted inline code).

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
  Bodies are full markdown: prose, lists, tables, and highlighted inline code all work
  inside them; titles render exactly as written because they are attribute values with no
  markdown or inline syntax.
- `Accordion` props: `title` (required), `type` = `accent` (default) or `default`,
  `defaultOpen` boolean (defaults to **true** — explicitly pass `{false}` to start closed).
  It holds one topic; stack multiple Accordion blocks for several. Its body takes the same
  full markdown as Alert bodies, and its title follows the same plain-text rule.
- Place content by visibility: required reasoning stays in prose or an open Alert;
  `!tooltip`, `!callout`, a closed Accordion, and a `<Tooltip>` hover note are for optional
  depth only.

## Note blocks (staff-note scaffolds)

A collapsible, tinted card with an icon and a colored header. The `type` picks the look
and default header label; the body is free-form markdown (prose, lists, tables). Blocks
are open by default and always print open.

````mdx
<NoteBlock type="teaching-tips">
<table className="note-block-table">
  <tbody>
    <tr><td>Facilitation</td><td>How to run the activity: grouping, roles, timing.</td></tr>
    <tr><td>Coaching</td><td>What to say or ask when students get stuck.</td></tr>
  </tbody>
</table>
</NoteBlock>

<NoteBlock type="answer-and-explanation" title="Answer and explanation — round 2">
State the answer, then explain why it is correct.
</NoteBlock>
````

- `type` = `teaching-tips` (blue, lightbulb) | `check-for-understanding` (green, circled
  check) | `answer-and-explanation` (dark green, message). Omit it for a colorless card.
  A mistyped value renders the block colorless with a visible warning line naming the bad
  value and the valid ones; the content still shows.
- For a label/content scaffold (like the teaching-tips example), use a raw HTML
  `<table className="note-block-table">` with one `<tr><td>label</td><td>content</td></tr>`
  per row. It renders borderless with no header row and no row shading (NoteBlock styles
  the class). A plain GFM pipe table also works but forces a header row.
- `title` (optional) overrides the type's default header label; plain text only, same rule
  as Alert/Accordion titles. A bare `<NoteBlock>` with neither a `type` label nor a
  `title` shows only the chevron.
- `defaultOpen` boolean (defaults to **true** — pass `{false}` to start collapsed). The
  header stays clickable either way, and print always renders the block open.
- The body takes the same full markdown as Alert/Accordion bodies (prose, lists, tables).
  Do not nest a NoteBlock inside another.

## Subsection titles (activity / Do Now headings)

A `<Subsection>` marker is a student-visible structural heading inside markdown element
content: it marks named activities within a section. It renders a horizontal rule, then the
type pill (a play triangle plus its label), then the title as a real `h2`, so it gets a
table-of-contents entry and an anchor like any other heading.

````mdx
<Subsection type="activity" title="Build a countdown timer"/>

### Start from the target value

The activity's own content follows, using `###` and deeper headings only.
````

- Attributes: exactly two. `type` = `activity` (green pill) or `do-now` (yellow pill); an
  unknown or missing type still renders the rule and the title as a plain `h2`, just without
  the pill (that is how you spot a typo in the live preview). `title` is plain text, same
  rule as Alert/Accordion titles.
- One marker per markdown element, as its first line: the staff notes page associates the
  element's instructional note with that one heading. Extra markers in the same element
  still render but stay non-structural (no notes-page row).
- Content inside an activity uses `###` and deeper: the subsection title already occupies
  the `h2` level, so a `##` after it would read as a sibling section, not part of the
  activity.
- Time is **not** an attribute. The staff-only ≈ time badge next to the pill comes from the
  host element's instructional note (set in the per-element note form); students never see
  it, and present mode shows no badge at all.
- Use markers only in markdown elements' content. In another element type's MDX fields
  (`question`, `hint`, `instruction`) a marker renders inline but is never structural: it
  never appears on the staff notes page.

## Terms and tooltips

Wrap a word or phrase in `<Tooltip tip="…">` when it deserves a definition or quick
clarification. The learner sees the note on hover, keyboard focus, or tap; the dashed
underline marks it (the same look as `!tooltip` inside code fences). Keep notes short: one
to three lines of inline text, anything longer belongs in an Accordion.

````mdx
The <Tooltip tip="An ordered list of **values** — supports `.push()`">array</Tooltip> is the workhorse structure.

- You can wrap a whole phrase: <Tooltip tip="A value that was never assigned to anything yet">*js undefined* means no value has been set for it yet</Tooltip>.
````

- The trigger stays inline inside one sentence: plain text, highlighted inline code, or an
  existing link all work. Never wrap block content (a heading, fence, or list of its own);
  a tooltip is optional depth, so keep required reasoning in prose.
- `tip` is a plain string attribute, so nothing in it compiles at write time: **bold**,
  `code`, and links written in the tip are rendered inside the bubble by the component at
  runtime instead. Same trap as Alert/Accordion titles — markdown stays inert in source.
- Code blocks are different: their annotations use `!tooltip` in fence meta, never a
  `<Tooltip>` tag (fence content is literal text and the JSX would show as characters).

## Images

- Plain markdown works: ![alt text](/api/files/<id>) renders in the page flow, best for
  small figures next to prose.
- For a zoomable viewer with explicit sizing use the component (click opens a full-screen
  dialog): `<ImageViewer url="/api/files/<id>" width="lg" />`. `width` and `height` accept:
  xxxs, xs, sm, md, lg, xl, xxl, xxxl, full.
- Only reference image URLs you were actually given; never invent file ids or paths.

## Embeds (iframes)

Intrinsic HTML elements (`div`, `iframe`) render as-is in lesson content — only unknown
PascalCase component names throw. To embed an external page or app:

````mdx
<div className="my-4">
  <iframe src="https://www.example.com/" title="Embedded content" width="100%" height="600px" loading="lazy" credentialless="true"></iframe>
</div>
````

- `credentialless="true"` is **required by default** on every iframe. This platform serves
  `Cross-Origin-Embedder-Policy: credentialless`, and embedded frames are expected to load
  without credentials; omitting the attribute breaks some embeds. The app's own embeds follow
  the same convention (`YouTube.tsx`, `StaticPreview.tsx`).
- Browsers default iframes to 300×150: always set explicit `width` and `height`.
- `title` is required (accessibility); write a short description of the embedded content.
- Add `allow` / `sandbox` attributes only when the target needs them (e.g. `allow="autoplay"`,
  `allow="cross-origin-isolated"`); keep the base embed minimal otherwise.

## Prose style

Structure:

- **Lesson-section markdown** (`data.code` of a markdown element): start at `##`; never use
  `#`. The lesson page already provides context above your element; a single element should
  not shout a top-level title. Headings are sentence case.
- **Short/inline fields** (multiple-choice question/options/explanations, free-response
  hint/answer, file-submission prompt, and any other field rendered inside a component):
  inline prose that must not open with a heading at all — no `#` of any level. The no-`#`
  half of the rule applies everywhere.
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

## Hard rules

1. Only the components listed in the Output contract exist (`Code` and `InlineCode` are
   produced automatically by code fences). Any other PascalCase tag throws a runtime error;
   a lowercase name like `<alert>` silently renders as a bare, unstyled DOM element.
2. No HTML comment line (`<!-- ... -->`) anywhere in the source — it is a compile error that
   breaks the whole element. If you ever need an internal note it must be `{/* text */}`;
   normally include no notes at all.
3. Use React attribute names on wrappers: `className=`, never `class=` or any other
   raw-HTML-style attribute — the styling is missing silently otherwise.
4. Close every code fence before the next heading, tag, or closing element (an unclosed
   fence swallows the rest of the document as code text), and put no backtick anywhere on a
   fence line: in its tag, flags, title, or label it stops being a fence at all.
5. Hike syntax is component-scoped: `!!`-prefixed headings and `!`/`!!`-prefixed fence metas
   only work inside Scrollycoding, Spotlight, or CodeWithTabs — outside they render as
   literal text. Every step has exactly one fence whose meta begins `!code`, and no prose
   precedes the first `## !!steps`.
6. Repeated same-name hike items need the double bang (`!!steps`, `!!tabs`): a single-bang
   repeat silently overwrites, leaving only the last item.
7. No highlighted inline code in step titles, tab labels, or any `title`/`tip` attribute —
   it renders literally as asterisks and backticks (or truncates a step title) with no error
   to catch it.
8. Annotate JSON with the `jsonc` tag, not `json` (JSON has no comments; in a `json` fence
   your annotation comments render as literal code lines). Old-style `{#mark}` brace
   annotations do nothing — use the `!` comment syntax.
9. `<Tooltip>` needs a non-empty `tip=` and wraps inline content only: never wrap block
   content, never nest tooltips.
10. One `<Subsection>` marker per markdown element, as its first line; spell the type
    exactly (`activity`, `do-now`) — a mistyped value degrades silently to a plain h2 with
    no pill.
11. Every iframe carries `credentialless="true"`, explicit `width`/`height`, and a
    descriptive `title`. Only reference image URLs you were actually given; never invent file
    ids or paths.
12. Section markdown starts at `##`; short/inline fields open with no heading at all — `#`
    is never used anywhere. Every paragraph is one continuous source line (a line break
    inside a paragraph becomes a visible `<br>`).

### Failure modes (what breaks rendering)

| If you write... | The learner sees... |
| --- | --- |
| A component outside the ten listed, or a lowercase name like `<alert>` | Runtime error box / bare unstyled element |
| An HTML comment line (`<!-- ... -->`) anywhere in the source | Compile error for the entire element |
| `class=` instead of `className=`, or any other raw-HTML-style attribute | Element renders unstyled; the styling is missing silently |
| An unclosed code fence | The rest of the document becomes code text |
| A plain fence where a step needs one tagged ```` !code ```` (see Scrollycoding rules) | Runtime error: invalid input for that step's code |
| `## !!steps` headings outside Scrollycoding/Spotlight | Literal "!!steps" text in a heading |
| Two same-name single-bang hike items (e.g. two `!tabs` fences instead of `!!tabs`) | Only the last one survives silently — use `!!` to accumulate them into an array |
| Old-style `{#mark}` brace annotations (CodeHike 0.x) | Plain comment text; nothing happens |
| Annotation comments inside a fence tagged `json` (JSON has no comments) | They render as literal code lines; use the tag `jsonc` to annotate JSON |
| A backtick anywhere on a code fence line (in its tag, flags, title, or label) | That line stops being a fence: it renders as prose and the next ``` opens a junk block that swallows everything until the next closing fence — inside Scrollycoding/Spotlight/CodeWithTabs this usually ends in an error box for the whole element |
| Highlighted inline code in a `title`/label attribute or in a `## !!steps` heading | Alert/Accordion/tab titles show literal asterisks and backticks; step titles are truncated (or empty) silently, with no error to catch it |
| A `<Tooltip>` without a non-empty `tip=` | The phrase renders as plain text: no underline, no bubble. Nothing errors — check your source |
| Nested tooltips (a `<Tooltip>` trigger wrapping another `<Tooltip>`) | Unsupported; behavior is undefined. Keep one note per phrase |
| Block content or a multi-paragraph explanation wrapped in a `<Tooltip>` | A cramped scrolling bubble that breaks out of the sentence; use an Accordion for that depth |
| A `<Subsection>` with a mistyped `type` (e.g. `activty`) | Rule + plain h2 title, no pill: nothing errors, so check the spelling (`activity`, `do-now`) in your source |

## Recipes / workflows

**Write a lesson section (a markdown element's `code`).**

1. Pick building blocks from the table above — the simplest thing that works; interactive
   layouts only when they teach better.
2. Read the example in `examples/` that matches what you are about to write before writing
   it (see Examples as fixtures).
3. Write per the reference sections: open with the concrete goal, keep one coherent example
   throughout, close with a small transfer exercise (Designing for the learner). For
   anything beyond mark/fold/collapse annotations, read
   [reference/annotations.md](reference/annotations.md) first.
4. Run the pre-output checklist before emitting.

**Write content for a short/inline field (question, hint, prompt).**

1. Same mechanics as above, minus section structure: inline prose that must not open with a
   heading; `#` is never used anywhere.
2. Keep it to what fits inside its component — optional depth belongs in an Accordion or the
   surrounding prose, not in a short field.

## Examples as fixtures

`examples/` holds worked MDX documents; each is verified by a gate (`npm run
verify:codehike-skill`) that compiles it through the exact production `mdxOptions` and renders
it without throwing — they must stay copy-paste-able into a real element's content field.

- [examples/annotated-code.mdx](examples/annotated-code.mdx): plain fences with meta flags,
  inline highlighted code, and the everyday annotations (mark, tooltip, fold, collapse,
  diff) in one page.
- [examples/steps-layouts.mdx](examples/steps-layouts.mdx): a complete Scrollycoding, a
  Spotlight, and a CodeWithTabs document showing the hike syntax end to end.
- [examples/terms.mdx](examples/terms.mdx): prose tooltips in their four documented
  placements: paragraph, list item with highlighted inline code in the trigger, GFM table
  cell, and Alert body.
- [examples/note-blocks.mdx](examples/note-blocks.mdx): a typed NoteBlock with a table
  body and a prose block with a `title` override.
- [examples/subsections.mdx](examples/subsections.mdx): both `<Subsection>` types: an
  Activity with its content under `###` headings, and a short Do Now beat.

Read the example that matches what you are about to write before writing it.

## Pre-output checklist

1. Raw MDX only — no preamble, no wrapping fence around the answer?
2. Every JSX tag is one of the ten listed components (PascalCase); wrappers use `className=`,
   not `class=`?
3. No HTML comment lines anywhere; internal notes as `{/* */}` or omitted entirely?
4. Every code fence closed, with no backtick on its fence line; annotated JSON tagged
   `jsonc`?
5. Hike syntax only inside Scrollycoding/Spotlight/CodeWithTabs: every step has exactly one
   `!code` fence, repeated items use the double bang, and no prose precedes the first
   `## !!steps`?
6. No highlighted inline code in any step title, tab label, or `title`/`tip` attribute?
7. Section markdown starts at `##`; short/inline fields open with no heading; every paragraph
   is one continuous source line?
8. `<Subsection>`: exactly one per element, first line, type spelled `activity`/`do-now`,
   content under `###`?
9. Every iframe has `credentialless="true"`, explicit size, and a `title`; every image URL was
   actually given to you?
10. Prose passes the anti-AI-slop rules: no banned vocabulary, no em dashes, straight quotes,
    no chatbot residue?

## Cross-reference map

- **lms-elements** — owns the 11 authorable element types' `data` shapes and minimal records.
  Hand off when the deliverable is a whole element record `{type, data}`; its CodeHike-routed
  fields delegate their content back here and embed your output verbatim in those fields.
- **lms-playground** — owns playground file sets (runtime choice, canonical package.json,
  Tailwind 4, boot rules) as SandpackFiles maps or flat-root zips. Hand off when the
  deliverable is a code-playground's files; its output lands in `code-playground.data.files.files`.
- **lms-backup** — owns whole backup JSON files: envelope, id/rank fabrication schemes,
  embedding records into files, the validation loop. It embeds your MDX verbatim into content
  fields; JSON escaping is its job.
