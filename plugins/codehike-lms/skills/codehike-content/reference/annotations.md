# Code annotations, complete reference

SKILL.md covers the basics. Use this file when you are actually annotating code blocks and
need exact syntax, query semantics, or behavior details. Everything here matches the
pipeline exactly; do not guess at variants.

## How annotation comments work

- An annotation is a **normal comment** in the block's language that starts with `!`:

  | Language family | Comment form for annotations |
  | --- | --- |
  | JS, TS, JSX/TSX, C, C++, Java, C#, Go, Rust, PHP, Swift, Kotlin, Dart, Scala | `// !name ...` |
  | Python, Ruby, Perl, YAML, shell scripts | `# !name ...` |
  | Lua, SQL | `-- !name ...` |
  | CSS | one-line block comment: `/* !name ... */` |
  | HTML, XML, markdown-in-HTML contexts | `<!-- !name -->` |
  | JSON | none exists. Tag the fence `jsonc` (or `json5`) instead if you must annotate it |

- The text after the name and range is the **query**. Its meaning depends on the handler
  (a color, tooltip text, a route, an index). Keep queries to one line: annotations live in
  single-line comments.
- Annotation comment lines are **removed from the rendered code**. Ordinary comments stay —
  so you can mix real explanation comments and annotation comments freely.
- With the `-p` meta flag on the fence, the prefix for that block is `!!` instead of `!`
  (write `// !!mark gold`). Use it when your sample code itself contains lines starting
  with `!`.

## The two shapes

### Block annotations (line ranges)

```text
// !name(fromLine:toLine) query     // numbers count from the line after this comment
// !name(start)                     ...
code in between                      ...
// !name(end)                       // pair form: no counting at all
// !name query                      // bare form: see placement rules below
```

- Line 1 is the line immediately below the annotation comment. Ranges are inclusive.
- A **bare** annotation (no range) placed on its own line applies to the next single line;
  the same bare annotation written as a trailing comment at the end of a code line
  (`const x = 5; // !mark gold`) applies to that line itself instead. Both placements are
  used in practice: trailing for one-line tints, standalone above when you also want it out
  of the way.
- Prefer start/end pairs when the annotated region sits mid-file; prefer ranges when you can
  count from the top of a small block. Avoid numeric column targeting entirely — regex form
  (below) survives edits without recounting.

### Inline annotations (token targeting)

```text
// !name[/pattern/flags] query      // standalone comment line above the target line
```

- The regex is matched in source order starting near the annotation; it attaches to each
  matched token range on that line (and beyond, with `m`).
- Regex flags: `g` = match every occurrence in search scope, `m` = keep searching past the
  first matching line. Capturing groups are supported and matter for `fold`.
- If you need several inline annotations on nearby tokens, stack their comment lines; each
  regex can target different text.

## The handlers

### mark — highlight a line or token
Scope: block + inline. Query: any CSS color (hex is safest); default sky blue
`rgb(14 165 233)`. Block form tints the whole line with a left accent border; inline form
renders the matched tokens as an outlined chip.

```js
// !mark gold
const result = compute(items);

// !mark[/compute/] pink
return compute(items);
```

### tooltip — hover explanation on a token
Scope: inline only. Query: plain text shown in a tooltip after a ~300ms hover delay; the
token gets a dashed underline. Write the query as a full sentence; it cannot contain
newlines.

```js
// !tooltip[/fetchJson/] "GETs the URL and parses the body as JSON; throws on non-2xx status."
const data = await fetchJson(url);
```

### pill — numbered badge on a token
Scope: inline only. Query: a number 1-6; the token gets a colored chip (colors cycle
green, teal, sky, violet, fuchsia, pink by index). Use to label steps or parts of code in
order ("first this, then that") without adding prose.

```js
// !pill[/fetchJson/] 1
const data = await fetchJson(url);
// !pill[/filter/] 2
return data.filter(isValid);
```

### fold — collapse matched content behind an expandable "..." button
Scope: inline only. The regex **must capture** what should hide (one capturing group).
The match is replaced by a `...` pill; clicking it shows the hidden text in place, clicking
again hides it again. Use for boilerplate you need present but want out of the way — long
class names, default option objects, repetitive markup.

```jsx
// !fold[/className="(.*?)"/gm]
function Card({ title }) {
  return (
    <div className="rounded border p-4 shadow-sm">
      <h3>{title}</h3>
    </div>
  );
}
```

If the regex matches nothing, the button shows empty content — test that your pattern
actually captures before relying on it. `/gm` is usually what you want for multi-line or
repeated matches.

### link — make a token navigate
Scope: inline only. Query: an app route path or URL (no angle brackets). The matched tokens
render underlined and open the target **in a new tab** when clicked. Use to point from code
to another lesson page, docs page, or reference.

```js
// !link[/useQuery/] /courses/abc123#unit-2-lessons
const [data] = useQuery();
```

### focus — dim everything except the lines you mark
Scope: block only; query unused. All non-focused lines drop to ~50% opacity, focused lines
get a subtle background, and the block auto-scrolls so the focused region is visible when it
changes. For long blocks where one region does all the teaching: annotate that region, let
the rest recede.

```js
// (long setup above)
// !focus(1:3)
function render(list) {
  return list.map(drawRow);
}
```

### mention — spotlight tagged lines from a prose link
Scope: block only; query = whitespace-separated ids (a line matches any mention whose id is in
the list). Renders nothing inside the code itself — it stamps each tagged line with its id(s)
so that `<CodeMentions>` wrappers can dim every non-matching line of the group while a matching
`hover:` markdown link in their prose is hovered or focused. Without both the wrapper and at
least one `hover:<id>` link inside it, this annotation has no visible effect; see SKILL.md's
"Code mentions (hover/focus highlight)" section for the full contract.

```js
// !mention fetchJson
const data = await fetchJson(url);
```

### ruler — colored vertical bar along a line range
Scope: block only. Query: optional `colorIndex secondToken` where colorIndex is 1-6 (same
green/teal/sky/violet/fuchsia/pink cycle as pill) and the second token shifts the bar
inward one notch, so two nested rulers read as different levels. Draws a thin rounded bar on
the left edge of each line in range; hovering the group tints it. Use to show scope or
grouping (which lines belong to which block), especially with `n` line numbers.

```js
// !ruler(1:3) 2
function outer() {
  inner();
}
```

### diff — mark added / removed lines git-style
Scope: block only. Query is exactly `+` (added, green tint and a `+` gutter mark) or `-`
(removed, red tint and a `-` mark). Use inside one fence to show an edit in place: the old
line with `-`, the new line right below it with `+`.

```js
// !diff -
const total = items.size;
// !diff +
const total = items.length;
```

### collapse — make a region expand/collapse behind a trigger line
Scope: block only. Query: `collapsed` makes the region start closed (omit it to start open).
The **first line of your range stays visible and becomes the clickable trigger** (chevron
icon on its left); every other line in the range hides when collapsed. Place the annotation
so the trigger is a natural header — usually a function signature — so what remains reads
like an API summary.

```js
// !collapse(1:4) collapsed
function parseConfig(text) {
  const obj = JSON.parse(text);
  validate(obj);
  return normalize(obj);
}
```

In the example above, line 1 (the signature) is the trigger; the body lines 2-4 hide when
closed and the closing brace (line 5) stays visible because it sits outside the range. To
hide that too — leaving only a bare signature — extend the range to cover the final line.

### callout — margin note attached to a token
Scope: inline only. Query: the text of a small bordered box that renders just below the
annotated line with an arrow pointing at the center of your matched tokens. Use for a
one-or-two-line aside about a specific value or flag, where interrupting the prose flow
would be worse.

```js
// !callout[/retries/] "Default is 0: without this option, failed requests never retry."
fetchJson("/api/items", { retries: 2 });
```

### className — apply utility classes to lines or tokens
Scope: block (wraps the line range in a `div`) + inline (`span` around matched tokens).
Query: any space-separated utility classes. The escape hatch for local styling that no other
handler covers. Keep it to small adjustments; page-level layout belongs in JSX wrappers like
`<div className="wide-content">` outside code blocks.

```js
// !className(1:2) opacity-50
const legacy = oldPath();
migrate(legacy);
```

## Flag-driven features (not annotations)

These are enabled by meta flags on the fence, not comments — see SKILL.md's flag table for
when to use them: `a` animates token-level transitions between steps in Scrollycoding/
Spotlight layouts; `n` adds a line-number gutter (pairs well with `ruler` and ranges); `w`
word-wraps long lines instead of scrolling horizontally; `c` adds the copy button. `-p`
switches the annotation prefix to `!!` for that block.

## Combination rules of thumb

- Annotations compose on the same line/block: a line can carry a `mark` range and its tokens
  can each have `tooltip`s or `pill`s.
- Never annotate code inside `CodeWithTabs` tabs, and never annotate `mermaid` fences —
  neither goes through the annotation pipeline; those comments would show as literal text.
- In step layouts (`!code` fences), annotations work exactly like in plain blocks; add `-a`
  so changes between steps animate instead of hard-swapping.
