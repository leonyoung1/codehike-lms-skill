---
name: lms-elements
description: "Author or inspect LMS2 element records — a complete minimal `{type, data}` record in backup-file shape for any of the 11 authorable types (markdown, free-response, multiple-choice, code-playground, image, youtube, file-submission, link, feedback, data-table, image-gen). Use when asked to produce or review a single element record. Not for: raw MDX prose (codehike-content), playground file sets (lms-playground), whole backup files (lms-backup)."
---

# LMS2 Element Records

One section per authorable type: the exact `data` shape, what renders where, and which
parser consumes each content field. Every example record in `examples/` is verified by a
gate (`npm run verify:lms-elements-skill`) — it must stay copy-paste-able into a real
backup file without breaking.

## Where your output goes

- **lms-backup** embeds this skill's records verbatim into backup JSON files (JSON
  escaping is its job). The `courseId`/`sectionId` in the examples are shared placeholders —
  lms-backup re-points them at the destination file's real roots when embedding.
- **Humans** read `data` directly; the top-level fields mirror what a real export carries,
  so a record from this skill is import-identical to one saved through the admin UI.

## Output contract

A complete minimal element record in backup-file shape:

```json
{
  "collection": "elements",
  "id": "<15 lowercase alphanumeric chars>",
  "type": "<one of the 11 types below>",
  "rank": "<zero-padded string, unique within its section>",
  "active": true,
  "access": 200,
  "courseId": "<placeholder or real course id>",
  "sectionId": "<placeholder or real section id>",
  "data": { "...type-specific fields..." }
}
```

Top-level notes:

- `access: 200` (students) is the app default for content records — element forms use it
  unoverridden. Never `0`: that is the course-level public default and would make a
  fabricated element public on import.
- `data` carries **type-specific fields only**. The admin form strips `type`, `active`,
  `access`, the submit flags, and `translations` out of it — do not duplicate them inside
  `data`.
- `updatedBy` (write-time author email) and `translations` are omitted from minimal
  records; both are optional. A translation entry mirrors the same fields under
  `translations.<lang>`.

### markdown

| field | required? | notes |
| --- | --- | --- |
| `code` | **yes** | CodeHike MDX (full pipeline). Section-level content: start at `##`, never `#`. Missing → the element renders empty. |

### free-response

Top-level flags: `isSubmitType: true`, `submissionRequired: true`.

| field | required? | notes |
| --- | --- | --- |
| `question` | **yes** | CodeHike MDX, inline prose (no heading root). |
| `answer` | no | Plain string for auto-grading (exact match after the case/whitespace options below); absent → no correct/incorrect indicator. Not rendered to students as markdown. |
| `caseSensitive` | no | bool, default `false`. Grading option. |
| `ignoreWhiteSpaces` | no | bool, default `false`. Grading option. |
| `rows` | no | Textarea row count, default `"1"`. |
| `hint` | no | CodeHike MDX shown in the Hint dialog. |
| `enableComments` | no | bool, default `true`. |
| `enableTeamView` | no | bool, default `false`; team members see each other's responses. |

### multiple-choice

Top-level flags: `isSubmitType: true`, `submissionRequired: true`.

| field | required? | notes |
| --- | --- | --- |
| `question` | **yes** | CodeHike MDX, inline prose (no heading root). |
| `options` | **yes** | Array of `{ text, explanation? }`, at least two. `text` is plain GFM (BasicMarkdown), not CodeHike. Missing → the renderer crashes; empty `text` renders a label-less choice. `explanation` shows after submission in quiz mode. |
| `correctAnswer` | no | Integer index into `options`, default `-1`. `-1` = no grading (any selection completes); `0..n-1` = quiz mode with change lockout. When ≥ 0 it must index an existing option, or the student can never be correct. |
| `hint` | no | CodeHike MDX shown in the Hint dialog. |

### code-playground

Top-level flags: `isSubmitType: true`, `submissionRequired: true`.

| field | required? | notes |
| --- | --- | --- |
| `files` | **yes** | Wrapper `{ activeFile, files, visibleFiles }`: `activeFile` a `/`-prefixed path, `files` the bare SandpackFiles map (`/`-prefixed keys → string contents), `visibleFiles` an array of paths that must exist in `files`. Missing → empty playground. **Authoring file sets is lms-playground's job — never hand-write them here.** |
| `environment` | no | `"static"` \| `"vite"`, default `"static"`. |
| `instruction` | no | CodeHike MDX rendered above the playground. |
| `hint` | no | CodeHike MDX shown in the Hint dialog. |
| `showFileExplorer` / `showEditor` / `showPreview` / `showTerminal` / `showPreviewErrors` | no | bools, defaults `false` / `true` / `true` / `false` / `true`. |
| `enableGithubImport` / `enableCodeAssistant` / `enableCodeAgent` / `showOpenInCodeSandbox` | no | bools, defaults `false` / `true` / `true` / `false`. |
| `splitPercent` | no | number, default `50`. Editor/preview split. |
| `readOnly` | no | bool, default `false`. |
| `width` / `height` | no | Widths / Heights keys (`full`, `xxxl`…`xxxs`), defaults `"xxl"` / `"md"`. |
| `direction` | no | `"horizontal"` \| `"vertical"`, default `"horizontal"`. |

### image

| field | required? | notes |
| --- | --- | --- |
| `url` | **yes** | Admin-file library URL or external https URL. Empty → no image renders (an empty box). Replace the example placeholder with a real upload before shipping content. |
| `description` | no | CodeHike MDX rendered beside the image. |
| `width` / `height` | no | Widths / Heights keys, default `"md"`. |
| `placement` | no | `top` \| `bottom` \| `left` \| `right`, default `right` (description side). Ignored at render time while the description contains a `<Subsection>` marker — layout is forced vertical (`bottom` respected, else top); see codehike-content. |

### youtube

| field | required? | notes |
| --- | --- | --- |
| `url` | **yes** | One of the three accepted shapes: `https://youtu.be/<id>`, `https://www.youtube.com/watch?v=<id>`, or a bare video id. Anything else is passed to the embed as-is and will break. |
| `description` | no | CodeHike MDX rendered beside the player. |
| `startTime` / `endTime` | no | Numbers (seconds) appended to the embed URL. |
| `width` / `height` / `placement` | no | As in image (including the subsection-marker vertical forcing), defaults `"md"` / `"md"` / `right`. |

### file-submission

Top-level flags: `isSubmitType: true`, `submissionRequired: true`.

| field | required? | notes |
| --- | --- | --- |
| `question` | **yes** | CodeHike MDX, inline prose (no heading root). |
| `hint` | no | CodeHike MDX. Renderer-supported; not exposed in the current admin form — safe to author, just know the UI won't edit it. |

### link

| field | required? | notes |
| --- | --- | --- |
| `url` | **yes** | External https URL (opens in a new tab) or an internal route path when `isInternal`. Missing → the element renders nothing at all. |
| `label` | no | Plain-text button caption (no markdown). Absent → icon-only button. |
| `description` | no | CodeHike MDX rendered beside the button. |
| `variant` | no | Default `"default"`. Allowed: `default`, `secondary`, `success`, `warning`, `destructive`, `info`, `ghost`, `outline`, `outline-default`, `outline-secondary`, `outline-success`, `outline-warning`, `outline-estructive` (sic — the form's own value). |
| `className` | no | Extra classes appended to the button. Not translatable. |
| `isInternal` | no | bool, default `false`. `true` = in-app navigation; `false`/absent = `window.open`. |
| `placement` | no | As in image (including the subsection-marker vertical forcing), default `right`. |

### feedback

Top-level flag: `isSubmitType: true` only (no `submissionRequired`).

| field | required? | notes |
| --- | --- | --- |
| `question` | **yes** | **Plain text — rendered raw with no markdown parsing at all.** Any markdown characters show literally. One short prompt sentence is the norm. |

### data-table

| field | required? | notes |
| --- | --- | --- |
| `csv` | **yes** | CSV with a header row, then one row per line. Missing → the renderer crashes. The parser handles quoted fields, escaped quotes (`""`), CRLF, and strips a leading BOM; columns are keyed by header name. |
| `enableFilters` | no | bool, default `false`. Per-column filter UI. |

### image-gen

| field | required? | notes |
| --- | --- | --- |
| `instructions` | **yes** | CodeHike MDX — the prompt shown to the student above the generate control. Inline prose (no heading root). |
| `placeholder` | no | String pre-filled into the prompt box, default empty. |

## Hard rules

1. Every CodeHike-routed field must be valid CodeHike MDX that compiles through the
   production pipeline — delegate content authoring to **codehike-content**. Section-level
   `markdown.code` starts at `##`; inline fields (questions, hints, descriptions,
   instructions) never open with a heading — except image/youtube/link `description`, which may
   open with a `<Subsection>` marker (structural; forces vertical layout); `#` is never used anywhere.
2. multiple-choice option `text`/`explanation` are plain GFM only (BasicMarkdown) — CodeHike
   components and hike syntax do not work there.
3. feedback's `question` renders as raw text — no markdown parsing at all.
4. Never hand-author code-playground file sets from this skill — delegate to
   **lms-playground**. The example's one-file map is a shape demo only, not a bootable
   template.
5. Admin form wrappers (`generate-markdown`, `generate-static-playground`,
   `generate-vite-playground`, `paste-json`) are never written to the DB under their own
   type — they save as their target type. Legacy `generate-markdown` records in old backup
   files render as markdown; treat them as `markdown` when editing.
6. multiple-choice: `options` is required (the renderer crashes without it), every option
   needs non-empty `text`, and `correctAnswer` ≥ 0 must index an existing option.
7. data-table: `csv` is required (the renderer crashes without it) and must be well-formed
   CSV with a header row.
8. youtube's `url` must match one of the three accepted shapes or the embed breaks.

## Recipes / workflows

**Author a new element record.**

1. Pick the type; copy `examples/<type>.json`.
2. Fill `data` per that type's table — delegate MDX content to codehike-content and any
   playground file set to lms-playground.
3. Set `id`, `rank`, and the parent pointers for your consumer (lms-backup owns the
   from-scratch fabrication schemes; a single record pasted into an existing section takes
   that section's real ids).
4. Run the pre-output checklist below.

**Inspect / validate an existing record.**

1. Check the top-level fields against the output contract (types, `access`, pointers).
2. Check every required `data` field for the type is present and non-empty where noted.
3. Check content-field parser validity: CodeHike fields compile; option text/explanation are
   plain GFM; feedback question has no markdown intent; csv parses with a header row.

## Examples as fixtures

`examples/` holds one bare JSON record per authorable type, named by the type string. Each is
gate-verified (`verify:lms-elements-skill`): all 11 compose into one synthetic section-kind
backup that validates clean through `validateBackup`, load-bearing shape assertions pass per
type, and every present CodeHike field compiles + renders through production `mdxOptions`.
The fixtures share placeholder pointers (`c00000000000001` / `s00000000000001`), ids
`e00000000000001`–`e00000000000011`, and ranks `"01"`–`"11"`.

## Pre-output checklist

1. Top-level: `collection`, `id` (15-char `[a-z0-9]`), `type`, `rank`, `active`, `access: 200`,
   `courseId`, `sectionId`, `data` all present with correct types?
2. Every required `data` field for the type present and non-empty where noted?
3. CodeHike fields valid MDX (no HTML comments, no unknown JSX tags)? Inline fields do not
   open with a heading; `markdown.code` starts at `##`?
4. multiple-choice: ≥ 2 options each with text; `correctAnswer` is `-1` or a valid index?
5. code-playground: `files` wrapper well-formed (activeFile/visibleFiles exist in the map),
   and the file set came from lms-playground, not hand-written here?

## Cross-reference map

- **codehike-content** — owns MDX mechanics, annotations, step layouts, prose style. Hand off
  every CodeHike-routed field's content to it; its output lands verbatim in those fields.
- **lms-playground** — owns playground file sets (runtime choice, canonical package.json,
  Tailwind 4, boot rules). Its output lands in `code-playground.data.files.files`.
- **lms-backup** — owns whole backup files: envelope, id/rank fabrication schemes, embedding
  records into files, the validation loop. This skill's records embed verbatim there; it
  re-points the placeholder roots.
