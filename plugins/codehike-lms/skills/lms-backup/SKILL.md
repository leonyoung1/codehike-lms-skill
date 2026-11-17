---
name: lms-backup
description: "LMS2 content backup JSON files — a flat record list + envelope for course/unit/lesson/section subtrees (the format the admin UI's Export menu downloads and the import dialog reads). Use when the deliverable is an LMS2 content backup file: editing one or creating a unit/lesson/section from scratch, including authoring element content inside it. Not for: raw MDX (codehike-content), playground file sets (lms-playground), single element records (lms-elements)."
---

# LMS2 Content Backup Files

## Where your output goes

The deliverable is a JSON **file** that an admin imports through LMS2's import dialog — either **create-new** (fresh ids generated, `courseId` remapped to the target course) or **in-place restore** (real-shaped ids matched against live records; tokens resolved through their stored `backupToken` or minted fresh — see Format reference). The file never touches PocketBase directly; import stamps `updatedBy` on every record it writes.

Inside records, element content is delegated — MDX prose follows codehike-content's mechanics, playground `data.files` maps follow lms-playground, per-type field shapes come from lms-elements. Your output is one bare JSON file with those artifacts embedded verbatim in the right fields.

## Output contract

- **One bare JSON document** per deliverable: no markdown fences, no BOM, no prose around it. (The validator tolerates both since D37/D38 — this skill does not produce them.)
- **Envelope**, always format version 2 (D49), two provenance shapes:
  - *Edited file / real export:* `{ "format": "lms-backup", "version": 2, "kind": "<course|unit|lesson|section>", "exportedAt": "<ISO date>", "source": { "id": "<root record id>", "name": "<root name>" }, "records": [...] }`
  - *From scratch:* the same keys **minus** `source` and `exportedAt`; a token root is allowed here only.
  Both may carry an optional envelope-level `"prune"` list — v2's only deletion path (Hard rule 7).
- **Records:** `{ "collection", "id", <PB fields verbatim> }` — the PocketBase response minus system fields (`created`, `updated`, `collectionId`, `collectionName`). `data` and `translations` pass through untouched.
- **Canonical record order** (what the exporter emits; keep it when editing): root first, then pre-order with siblings rank-sorted (unit → its lessons → each lesson's sections → each section's elements), notes last grouped by tier unit → lesson → section → element.

## Format reference

The canonical contract is `docs/specs/content-backup-restore.md` §3.1/§3.2 (decisions D7–D52) — this section is a working summary, not the contract. When in doubt, the spec and `validateBackup` (`src/lib/pb/backupFormat.ts`) win.

**Format v2 — ids and deletion.** Your files are always version 2; restore semantics follow the file's version (D49):

- **Ids are two-class (D45/D51).** A real-shaped id (`^[a-z0-9]{15}$`) is a PocketBase id: under in-place restore it matches/updates that exact live record. Anything else is a *token* — a file-local label the app replaces with a freshly minted real id on import, remapping every pointer (parent pointers, note targets) that references it. Real-shaped ids appear only on records you know exist live (from an export); **never fabricate one** — fabricated real-shaped ids persist as primary keys and collide across agent-authored files.
- **Deletion is explicit (D47).** Under v2 restore, absence from `records[]` does NOT delete a live record; the only deletion path is the envelope's optional `"prune"` list (`[{ "collection", "id" }]`, id = real id or token). Deleting something means removing it from `records[]` **and** adding its prune entry.
- **`backupToken` is app-stamped, not authored (D46).** A record created from a token carries that token in a PB field called `backupToken`; exports include it verbatim like any other field. Never set or clear it yourself — preserve it when editing so future restores can resolve the record's token.

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

1. **The root record's id is immutable when editing.** In-place restore matches records by id, and validation rule 8 cross-checks `source.id` against the root record's id — changing either breaks the file. From-scratch files carry a token root instead of a real id (D45).
2. **Your output is always format version 2** (`"format": "lms-backup"` frozen; `"version": 2`). The exporter still emits v1, but any edit bumps a file to v2 — so anything you deliver is v2 with patch-default restore semantics (D49). Never emit or downgrade other versions.
3. **Parent pointers resolve within the file**, except: every non-course record's `courseId` (external — remapped on import), and a tier-kind root's own parent pointer (`unitId` on a lesson-kind root, `lessonId` on a section-kind root) pointing outside the file. From-scratch files may use real destination ids or tokens there; edited files keep whatever the export had.
4. **(parent, rank) is unique per sibling group** — units by `courseId`, sections by `lessonId`, elements by `sectionId`. Lessons have no PB unique index on (unitId, rank), but keep their ranks sane anyway: import sorts by them.
5. **Ranks are lexicographic strings.** Never start one with `~` — that prefix is the reserved temp namespace for in-app drag reordering.
6. **Ids are two-class (D45).** Real-shaped (`^[a-z0-9]{15}$`) only for records you know exist live — when editing, preserve original ids (in-place restore depends on them). New records get a *token*: `<kind letter>-<6 random [a-z0-9] chars>` — `u-` / `l-` / `s-` / `e-`, `n-` for notes, `c-` for an unknown destination course. **Random only, never sequential counters** (counters make cross-file token reuse predictable). Never fabricate a real-shaped id: it would persist as a PocketBase primary key and collide across agent-authored files. When editing, preserve existing tokens too — they are the record's identity across edit rounds; renaming one orphans its live record into a duplicate.
7. **Deletion = remove from `records[]` AND add a `prune` entry** (`{ "collection", "id" }`, real id or token). Under v2 restore, absence alone does not delete (D47) — and deletion is unrecoverable, so it requires this positive authorization. One prune entry per deleted root suffices: PocketBase cascades children and notes. A record may not appear in both `records[]` and `prune[]`; the root record may never be pruned.
8. **Notes:** exactly one tier target set per note; at most one note per record per tier; non-empty `note` text.
9. **A rename edits `name` AND every `translations.<lang>.name`.** Same for any other translated field you change — a half-translated rename leaves stale strings in the UI.
10. **Output is bare JSON** — no fences, no BOM, no prose around it.
11. **Don't fabricate content shapes.** Element record fields come from lms-elements; MDX from codehike-content; playground file maps from lms-playground. This skill owns the envelope, ids/ranks/pointers, and JSON escaping — not the content inside `data`.

## Recipes / workflows

Simplest first. All recipes end with the validation loop below before delivery.

1. **Rename a unit / lesson / section.** Load → find the record by id → set `name` plus every `translations.<lang>.name` → save bare JSON. Nothing else changes; ids and ranks stay put.
2. **Reorder siblings.** Change only the `rank` strings within one sibling group (same parent). Keep width consistent (`"01"`…`"12"`); lexicographic order is display order. Renumber the whole group rather than inserting fractional ranks — `~`-prefixed temp ranks belong to in-app drags, not files.
3. **Move a subtree** (e.g. a lesson into another unit within the same file). Change the moved root's parent pointer (`unitId`) and give it a rank that slots it into the destination group. Everything below moves with it — no other edits needed. Cross-course moves are out of scope for an in-file edit: export the subtree as its own tier file and import create-new into the target course.
4. **Delete a subtree.** Remove the record *and* everything reachable under it (its lessons / sections / elements), plus any notes targeting removed records — and add one `prune` entry for the deleted root (`{ "collection", "id" }`, real id or token). PocketBase cascades children and notes, so descendants need no prune entries of their own. Left-behind children become orphans; left-behind notes fail validation — both are caught by rule 4 of `validateBackup`. Forgetting the prune entry is the safe failure under v2: the record simply survives (D47).
5. **Add a record from scratch** to an existing file. Fresh token id per Hard rule 6 (never a fabricated real-shaped id), parent pointer to an in-file record, rank appended after the group's last sibling, `active: true`, `access` matching its siblings. Element records need a valid `data` per lms-elements — delegate the content fields there.
6. **Create a whole tier file from scratch.** Envelope without `source` / `exportedAt` at version 2; root with a token id and rank `"01"`; external parent pointers = the real target course/unit/lesson ids when known, else tokens (`c-…` for every record's `courseId`, plus `u-…` / `l-…` for a lesson-/section-kind root's own parent). Build children pre-order with token ids. The kitchen-sink example is the reference shape.

**Validation loop (dual).**
- *Fast:* run the shared validator — the same code as `/dev/backup` and the import dialog: `validateBackup` in `src/lib/pb/backupFormat.ts`. It collects **all** problems, not fail-fast; each is a machine-readable `path — message` line (e.g. `records[14].rank`). For an arbitrary file, paste it into `/dev/backup` in the dev app and read `{ ok: true }` / `{ ok: false, problems }`.
- *Final:* import into a throwaway course on the test instance via the UI (create-new), verify rendering — sections load, playgrounds boot, notes appear in present mode — then delete the scratch content. Browser-tool note: MCP `fill`-style value injection does **not** fire React 19's onChange on the import dialog's textarea; drive the drop zone's file input with a real file upload instead (Phase 3 prep-B verification note).

## Examples as fixtures

Five files in `examples/`, pinned by shape in `src/lib/pb/skill-content.test.ts` (`npm run verify:lms-backup-skill`):

- **`unit-kitchen-sink.json`** — hand-crafted from scratch (S10 envelope at version 2, D45 token ids on every record including the root and the destination `courseId`, root rank `"01"`). All 12 element types, notes at all four tiers, es/pt translations. The reference for creating a file with no source export.
- **Four real UI exports from one small course** on the test instance — "Innovation Showcase Preparation", exported 2026-11-05 through the admin Export menus (filenames follow §3.1's download convention):
  - `course-innovation-showcase-preparation-2026-11-05.json` (whole course: 3 units / 13 lessons / 21 sections / 80 elements)
  - `unit-innovation-showcase-preparation-information-2026-11-05.json`
  - `lesson-innovation-showcase-preparation-logistics-2026-11-05.json`
  - `section-innovation-showcase-preparation-week-2-daily-agendas-2026-11-05.json`

  These show what real records look like: mudder base62 ids, verbatim `updatedBy` (on the record *and* inside some element `data`), `translations: null` where unset, and `source.pbUrl` in the envelope. Bumped to version 2 as edited exports (D49), with three documented hand-edits per S11:

  - **Hand-added note** (the source course had no notes): one element-tier note "Daily agendas" targeting markdown element `x379qurs07o1ts2`, appended to **all four** files under the *same token id* — that element sits inside every exported subtree, so a real DB note would appear in all of them.
  - **es translation** (the source course had no translations): `{ "es": { "name": "Información" } }` on unit `k8i5n0jiatdn9rn` ("Information") in the **course** and **unit** files, the only two that contain that record.
  - **Prune demo** (section file only): element `32p45l06whce19i` was removed from `records[]` and enumerated in the envelope's `prune` list — demonstrating D47's explicit-deletion shape. This is an intentional divergence: the other three files still contain that element, because under v2 each file is its own patch (omission ≠ deletion).

## Pre-output checklist

1. Envelope: `format` `"lms-backup"`, `version` 2, `kind` matches the root record's collection; `source` / `exportedAt` present iff this is an edited real export — and then `source.id` equals the root id (real-shaped) while a from-scratch file may carry a token root.
2. Exactly one root of the right collection; every other record reachable from it via parent pointers (or a note targeting an in-file record).
3. Ids two-class (D45): real-shaped only for records known live (originals preserved when editing), tokens `<letter>-<6 random>` for new records, unique across the file — no fabricated real-shaped ids anywhere.
4. Deletions: every removed record's subtree root has a matching `prune` entry; nothing appears in both `records[]` and `prune[]`; the root is not pruned.
5. Ranks: non-empty, no `~` prefix, consistent width per sibling group; (parent, rank) unique where PB enforces it.
6. Notes: exactly one tier target each; at most one note per record per tier; non-empty text.
7. Renames touched every `translations.<lang>.name`; existing tokens and `backupToken` fields preserved verbatim.
8. Element content delegated correctly — MDX follows codehike-content (no HTML comments, scoped prose), playground maps boot per lms-playground, data shapes match lms-elements.
9. Bare JSON output: no fences, no BOM, no surrounding prose; `validateBackup` clean before delivery.

## Cross-reference map

- **codehike-content** — the MDX inside `data.code`, questions/hints/instructions/descriptions, and note text.
- **lms-playground** — the SandpackFiles maps inside code-playground `data.files`.
- **lms-elements** — per-type element record shapes (the 11 authorable types).
