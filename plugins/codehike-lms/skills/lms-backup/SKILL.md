---
name: lms-backup
description: "LMS2 content backup JSON files — a flat record list + envelope for course/unit/lesson/section subtrees (the format the admin UI's Export menu downloads and the import dialog reads). Use when the deliverable is an LMS2 content backup file: editing one or creating a unit/lesson/section from scratch, including authoring element content inside it. Not for: raw MDX (codehike-content), playground file sets (lms-playground), single element records (lms-elements)."
---

# LMS2 Content Backup Files

## Where your output goes

The deliverable is a JSON **file** that an admin imports through LMS2's import dialog — either **create-new** (fresh ids generated, `courseId` remapped to the target course) or **in-place restore** (records matched by id against existing content). The file never touches PocketBase directly; import stamps `updatedBy` on every record it writes.

Inside records, element content is delegated — MDX prose follows codehike-content's mechanics, playground `data.files` maps follow lms-playground, per-type field shapes come from lms-elements. Your output is one bare JSON file with those artifacts embedded verbatim in the right fields.

## Output contract

- **One bare JSON document** per deliverable: no markdown fences, no BOM, no prose around it. (The validator tolerates both since D37/D38 — this skill does not produce them.)
- **Envelope**, two shapes:
  - *Edited file / real export:* `{ "format": "lms-backup", "version": 1, "kind": "<course|unit|lesson|section>", "exportedAt": "<ISO date>", "source": { "id": "<root record id>", "name": "<root name>" }, "records": [...] }`
  - *From scratch:* the same keys **minus** `source` and `exportedAt`.
- **Records:** `{ "collection", "id", <PB fields verbatim> }` — the PocketBase response minus system fields (`created`, `updated`, `collectionId`, `collectionName`). `data` and `translations` pass through untouched.
- **Canonical record order** (what the exporter emits; keep it when editing): root first, then pre-order with siblings rank-sorted (unit → its lessons → each lesson's sections → each section's elements), notes last grouped by tier unit → lesson → section → element.

## Format reference

The canonical contract is `docs/specs/content-backup-restore.md` §3.1/§3.2 (decisions D7–D40) — this section is a working summary, not the contract. When in doubt, the spec and `validateBackup` (`src/lib/pb/backupFormat.ts`) win.

**Contents per kind** — each file holds one subtree, active *and* inactive records:

| kind | records included |
| --- | --- |
| `course` | the course + all its units / lessons / sections / elements + notes on any of them |
| `unit` | that unit + its lessons / sections / elements + in-scope notes |
| `lesson` | that lesson + its sections / elements + in-scope notes |
| `section` | that section + its elements + in-scope notes (the smallest exportable unit — there is no single-element kind) |

- Every non-course record carries `courseId` verbatim; import remaps it to the target course.
- **Not included:** classes, enrollments, teams, file-store contents.
- `sections.objectives` (optional MDX text) passes through verbatim like any other content field.

**Element types — pointer table.** Record shapes live in lms-elements; this table only routes where each type's *content* comes from:

| type | content fields | domain |
| --- | --- | --- |
| `markdown` | `data.code` (full pipeline, starts at `##`) | codehike-content |
| `multiple-choice` | `question`, `hint` (MDX inline prose); options are plain GFM | codehike-content for the MDX fields |
| `free-response` | `question`, `hint` (MDX inline prose) | codehike-content |
| `code-playground` | `data.files` wrapper `{activeFile, files, visibleFiles}` around a SandpackFiles map; `instruction`, `hint` are MDX | lms-playground for the file set |
| `image` / `youtube` / `link` | URL + optional `description` (MDX) / `label` | codehike-content for descriptions |
| `file-submission` | `question`, `hint` (MDX inline prose) | codehike-content |
| `feedback` | `question` — **plain text, no markdown** | — |
| `data-table` | `data.csv` with a header row | — |
| `image-gen` | `instructions` (MDX inline prose) | codehike-content |

**Instructional notes.** Collection `instructional_notes`; one record is `{ collection, id, courseId, <exactly one of elementId | sectionId | lessonId | unitId>, title, time (seconds), active, note }`. Four tiers — unit / lesson / section / element — with exactly one target set per note and at most one note per record per tier. `note` content is CodeHike MDX written for the **instructor** audience (codehike-content's mechanics apply; it is staff-facing prose, not student-facing callouts).

## Hard rules

1. **The root record's id is immutable when editing.** In-place restore matches records by id, and validation rule 8 cross-checks `source.id` against the root record's id — changing either breaks the file.
2. **Never change envelope `format` / `version`.** `"lms-backup"` / `1`.
3. **Parent pointers resolve within the file**, except: every non-course record's `courseId` (external — remapped on import), and a tier-kind root's own parent pointer (`unitId` on a lesson-kind root, `lessonId` on a section-kind root) pointing outside the file. From-scratch files may use real ids or S7 placeholders there; edited files keep whatever the export had.
4. **(parent, rank) is unique per sibling group** — units by `courseId`, sections by `lessonId`, elements by `sectionId`. Lessons have no PB unique index on (unitId, rank), but keep their ranks sane anyway: import sorts by them.
5. **Ranks are lexicographic strings.** Never start one with `~` — that prefix is the reserved temp namespace for in-app drag reordering.
6. **Ids:** 15-char `[a-z0-9]`, unique across the file. When editing, preserve original ids (in-place restore depends on them). From scratch: use the fabrication scheme `<kind letter><zero-padded counter to 14 digits>` — `c…` / `u…` / `l…` / `s…` / `e…` / `n…` for notes. Never copy real ids from another file or a live record into a from-scratch file.
7. **Notes:** exactly one tier target set per note; at most one note per record per tier; non-empty `note` text.
8. **A rename edits `name` AND every `translations.<lang>.name`.** Same for any other translated field you change — a half-translated rename leaves stale strings in the UI.
9. **Output is bare JSON** — no fences, no BOM, no prose around it.
10. **Don't fabricate content shapes.** Element record fields come from lms-elements; MDX from codehike-content; playground file maps from lms-playground. This skill owns the envelope, ids/ranks/pointers, and JSON escaping — not the content inside `data`.

## Recipes / workflows

Simplest first. All recipes end with the validation loop below before delivery.

1. **Rename a unit / lesson / section.** Load → find the record by id → set `name` plus every `translations.<lang>.name` → save bare JSON. Nothing else changes; ids and ranks stay put.
2. **Reorder siblings.** Change only the `rank` strings within one sibling group (same parent). Keep width consistent (`"01"`…`"12"`); lexicographic order is display order. Renumber the whole group rather than inserting fractional ranks — `~`-prefixed temp ranks belong to in-app drags, not files.
3. **Move a subtree** (e.g. a lesson into another unit within the same file). Change the moved root's parent pointer (`unitId`) and give it a rank that slots it into the destination group. Everything below moves with it — no other edits needed. Cross-course moves are out of scope for an in-file edit: export the subtree as its own tier file and import create-new into the target course.
4. **Delete a subtree.** Remove the record *and* everything reachable under it (its lessons / sections / elements), plus any notes targeting removed records. Left-behind children become orphans; left-behind notes fail validation — both are caught by rule 4 of `validateBackup`.
5. **Add a record from scratch** to an existing file. Fresh id per the S7 scheme (or any non-colliding 15-char `[a-z0-9]`), parent pointer to an in-file record, rank appended after the group's last sibling, `active: true`, `access` matching its siblings. Element records need a valid `data` per lms-elements — delegate the content fields there.
6. **Create a whole tier file from scratch.** Envelope without `source` / `exportedAt`; root with an S7 id and rank `"01"`; external parent pointers = the real target course/unit/lesson ids when known, else S7 placeholders (`c…` for every record's `courseId`, plus `u…` / `l…` for a lesson-/section-kind root's own parent). Build children pre-order. The kitchen-sink example is the reference shape.

**Validation loop (dual).**
- *Fast:* run the shared validator — the same code as `/dev/backup` and the import dialog: `validateBackup` in `src/lib/pb/backupFormat.ts`. It collects **all** problems, not fail-fast; each is a machine-readable `path — message` line (e.g. `records[14].rank`). For an arbitrary file, paste it into `/dev/backup` in the dev app and read `{ ok: true }` / `{ ok: false, problems }`.
- *Final:* import into a throwaway course on the test instance via the UI (create-new), verify rendering — sections load, playgrounds boot, notes appear in present mode — then delete the scratch content. Browser-tool note: MCP `fill`-style value injection does **not** fire React 19's onChange on the import dialog's textarea; drive the drop zone's file input with a real file upload instead (Phase 3 prep-B verification note).

## Examples as fixtures

Five files in `examples/`, pinned by shape in `src/lib/pb/skill-content.test.ts` (`npm run verify:lms-backup-skill`):

- **`unit-kitchen-sink.json`** — hand-crafted from scratch (S10 envelope, S7 ids, root rank `"01"`). All 11 element types, notes at all four tiers, es/pt translations. The reference for creating a file with no source export.
- **Four real UI exports from one small course** on the test instance — "Innovation Showcase Preparation", exported 2026-11-05 through the admin Export menus (filenames follow §3.1's download convention):
  - `course-innovation-showcase-preparation-2026-11-05.json` (whole course: 3 units / 13 lessons / 21 sections / 80 elements)
  - `unit-innovation-showcase-preparation-information-2026-11-05.json`
  - `lesson-innovation-showcase-preparation-logistics-2026-11-05.json`
  - `section-innovation-showcase-preparation-week-2-daily-agendas-2026-11-05.json`

  These show what real records look like: mudder base62 ids, verbatim `updatedBy` (on the record *and* inside some element `data`), `translations: null` where unset, and `source.pbUrl` in the envelope. The four files are mutually consistent — every record of a smaller file appears unchanged in the larger ones that contain its subtree.

  **Hand-added per S11** (the source course had no notes or translations; added so the real exports demonstrate both shapes): one element-tier note `n00000000000099` ("Daily agendas", targeting markdown element `x379qurs07o1ts2`) appended to **all four** files — that element sits inside every exported subtree, so a real DB note would appear in all of them — and an es translation (`{ "es": { "name": "Información" } }`) on unit `k8i5n0jiatdn9rn` ("Information") in the **course** and **unit** files, the only two that contain that record. The note's id follows the S7 scheme so it is recognizable as fabricated.

## Pre-output checklist

1. Envelope: `format` `"lms-backup"`, `version` 1, `kind` matches the root record's collection; `source` / `exportedAt` present iff this is an edited real export — and then `source.id` equals the root id.
2. Exactly one root of the right collection; every other record reachable from it via parent pointers (or a note targeting an in-file record).
3. Ids: 15-char `[a-z0-9]`, unique across the file, originals preserved when editing.
4. Ranks: non-empty, no `~` prefix, consistent width per sibling group; (parent, rank) unique where PB enforces it.
5. Notes: exactly one tier target each; at most one note per record per tier; non-empty text.
6. Renames touched every `translations.<lang>.name`.
7. Element content delegated correctly — MDX follows codehike-content (no HTML comments, scoped prose), playground maps boot per lms-playground, data shapes match lms-elements.
8. Bare JSON output: no fences, no BOM, no surrounding prose; `validateBackup` clean before delivery.

## Cross-reference map

- **codehike-content** — the MDX inside `data.code`, questions/hints/instructions/descriptions, and note text.
- **lms-playground** — the SandpackFiles maps inside code-playground `data.files`.
- **lms-elements** — per-type element record shapes (the 11 authorable types).
