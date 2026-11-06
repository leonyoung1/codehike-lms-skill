---
name: lms-playground
description: "Build runnable LMS2 code playgrounds as file sets for the in-browser WebContainers sandbox, either static or vite. Covers choosing between the two runtimes, canonical package.json / dev-port requirements, Tailwind 4 setup in both, and outputting as a SandpackFiles JSON map or an importable flat-root zip. Use when asked to create, generate, scaffold, or fix a code playground — exercise sandboxes, p5.js/Phaser demos, React/Svelte/Vite projects, static HTML+JS interactive pages — for LMS2 lesson content. Not for: raw CodeHike MDX prose (codehike-content), element records `{type, data}` (lms-elements), whole backup JSON files (lms-backup); and not for editing the playground runtime itself (`src/lms/playground`)."
---

# LMS2 Code Playground Builder

You are producing a complete set of project files for an in-browser code playground: students see them in an editor and watch a live preview. The platform boots every preview the same way — `pnpm install`, then `pnpm run dev-port` with the assigned port injected as the `$PORT` environment variable. Your job is to make files that **boot on first load** and teach well.

You have no access to the app or its source; this skill is your complete reference for what will work.

## Where your output goes

Your file set lands in a code-playground element — you emit the bare map or zip only:

- The SandpackFiles map goes into `code-playground.data.files.files`, inside the
  `{ activeFile, files, visibleFiles }` wrapper; the zip is the human-import form of that same
  file set. The wrapper's full field set is documented by **lms-elements** (its code-playground
  section) — this skill owns only the bare map itself.
- Standalone consumers take your output as-is: machine consumers parse the raw JSON wholesale and
  assign it straight into Sandpack's `files.files`; humans import the flat-root zip through LMS2's
  ImportButton.
- **lms-elements** delegates every code-playground file set to you and places your map in the
  wrapper of the record it authors; **lms-backup** embeds that record verbatim into backup JSON
  files — JSON escaping is its job, not yours.

## Output contract

The JSON form exists for machine consumers that parse your raw output wholesale and assign it straight into Sandpack's `files.files`. A bare map of `/`-prefixed paths to string contents is the only shape they accept: no markdown fences, no prose, no wrapper keys. The zip form exists for human import via LMS2's ImportButton: flat root, no dotfiles or lockfiles, explicit `dev-port`.

### SandpackFiles JSON (machine-consumed)

A single bare JSON object mapping file paths to string contents. Nothing else in the output — no markdown fences, no prose before or after, no wrapper keys:

```json
{
  "/main.js": "…",
  "/index.html": "<!DOCTYPE html>…"
}
```

(the fence above only exists so this document can display it — your output is the object itself)

- Every key starts with `/` (a bare `main.js` becomes a garbage file in the sandbox). Values are always strings.
- **Primary student-editable file first** — consumers use the first key as the active editor tab.
- Always ship an explicit `dev-port` script; no consumer rewrites your package.json for you.
- Creating from scratch: output the complete tree.
- Editing existing files (you were given the current map): changed/added files only is fine — consumers merge into the current set. To delete a file, or to rebuild wholesale, output the full updated tree instead.

### ZIP (human-imported)

A flat-rooted archive named `<slug>.zip` (`bouncing-ball.zip`, not `project/bouncing-ball.zip`) containing exactly the project files:

- No top-level folder — importers don't strip one and a stray root folder breaks every path.
- No dotfiles, no `node_modules/`, no build output, no lockfiles.
- Same package.json rules as JSON (explicit `dev-port` always; the importer only auto-injects it for vite/astro zips).
- Put the primary student-editable file first in the archive — it becomes the active tab.

Planned (not yet wired): honoai's playground generation will adopt this skill as its single source, replacing the hardcoded prompts it uses today. When that lands, the rules above are the contract — don't relax them (extra keys, pretty-printed wrappers) without checking LMS2's `CodeAgent.tsx` and `GenStaticPlaygroundType.tsx`.

## Choosing a runtime

| | Static | Vite |
| --- | --- | --- |
| Best for | Plain HTML/CSS/JS, p5.js, Phaser — anything that can run from CDN ESM imports | React, Svelte, or anything needing bundling, TypeScript compilation, or npm packages |
| Why it loads fast | Tiny dependency list (one dev server), no build step in the sandbox | Fast too, but only while dependencies stay lean |
| Package rule | **CDN only** — imports are relative paths or `https://cdn.jsdelivr.net/...` URLs. If a library has no clean ESM CDN form, don't fight it: use Vite instead | Declare real dependencies in `package.json`; keep the list minimal (the sandbox installs on every load) |

Default to static for anything that is fundamentally an HTML+JS page; reach for vite when you need a framework or a build step.

## Static playgrounds

The `package.json` must be exactly this shape — one dev server, nothing else:

```json
{
  "scripts": {
    "dev": "servor --reload",
    "dev-port": "servor . index.html $PORT --reload",
    "start": "servor --reload"
  },
  "dependencies": {
    "servor": "^4.0.2"
  },
  "devDependencies": {}
}
```

`$PORT` is set by the platform — never hardcode a port, and don't add scripts or dependencies beyond this shape. The server is **servor**, not web-dev-server: WDS ran out of memory on school Chromebooks (8 GB RAM) and left previews spinning forever; servor boots fine there.

**Hard rules:**
- **No bare specifiers, no extensionless imports — there is no resolver, only the browser.** Imports are relative paths *with* the file extension (`./main.js`) or jsDelivr ESM URLs (`https://cdn.jsdelivr.net/npm/<pkg>@<version>/+esm`).
- **Multi-page = real `.html` files**, linked with plain `<a href="page2.html">`. No history-API routing (`pushState`/`replaceState`, extensionless routes).
- **Expect `[servor] listening for file changes` in every preview's console** — servor's live-reload client; harmless, don't suppress it.

**Tailwind 4:** one script tag in `index.html`, no install, no config:

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

Pin exact versions you have checked work (e.g. `p5@2.3.4`); Tailwind uses the `@4` tag as above.

p5.js from a CDN ESM import, instance mode:

```js
import p5 from "https://cdn.jsdelivr.net/npm/p5@2.3.4/+esm";
new p5((p) => { /* p.setup / p.draw */ }, document.body);
```

## Vite playgrounds

Keep it lean — every extra package is a slower sandbox install:

- Dependencies: `react`, `react-dom` (or the one framework you actually need). Dev dependencies: `vite`, `@vitejs/plugin-react` (only with React), `tailwindcss`, `@tailwindcss/vite`. Nothing else.
- **Plain JavaScript by default** (`main.jsx`/`App.jsx`). Use TypeScript only when explicitly asked — and then also add `tsconfig.json` plus the matching `@types/*` packages.
- Tailwind 4 = register the plugin in `vite.config.js` and one CSS import:

```js
import tailwindcss from "@tailwindcss/vite";
export default defineConfig({ plugins: [react(), tailwindcss()] });
```

```css
@import "tailwindcss";
```

- Scripts follow the same rule as static — `dev-port` must respect `$PORT`:

```json
{ "scripts": { "dev": "vite", "dev-port": "vite --port $PORT" } }
```

## Quality bar

- **Tailwind 4 styling in every playground**, both runtimes (browser tag / vite plugin) — no hand-rolled utility CSS.
- This is teaching material: meaningful names, short comments at non-obvious points, and structure students can extend (clear hooks for the exercise).
- No giant blob files: split by purpose (`main.js`, `physics.js`…; `App.jsx`, `components/…`). One job per file.
- Must boot on first load: no missing imports, no typo'd specifiers, correct `dev-port`. If you can't be sure a library loads from CDN ESM, choose vite and declare it as a dependency instead.

## Hard rules

1. Every key of a JSON map starts with `/` and every value is a string — a bare `main.js` becomes
   a garbage file in the sandbox.
2. Static imports resolve without a bundler: relative paths *with* the file extension or jsDelivr
   ESM URLs only — no bare specifiers, no extensionless imports (there is no resolver, only the
   browser).
3. Always ship an explicit `dev-port` script that respects `$PORT`; never hardcode a port — no
   consumer rewrites your package.json for you, and the zip importer only auto-injects it for
   vite/astro zips.
4. The static `package.json` is exactly the canonical shape (one dev server, nothing else) — don't
   add scripts or dependencies beyond it; the vite set stays lean because the sandbox installs on
   every load.
5. JSON output is one bare map and nothing else: no markdown fences, no prose before or after, no
   wrapper keys — machine consumers parse your raw output wholesale.
6. Zips are flat-rooted with exactly the project files: no top-level folder (importers don't strip
   one), no dotfiles, no `node_modules/`, no build output, no lockfiles.
7. The primary student-editable file comes first — first key in a JSON map, first entry in a zip;
   consumers make it the active editor tab.
8. Multi-page static playgrounds are real `.html` files linked with plain `<a href>` — no
   history-API routing (`pushState`/`replaceState`, extensionless routes).
9. `/changes.md` belongs in JSON edit output only: stripped from the sandbox and rendered as your
   explanation (missing → your answer renders empty); never in a zip, where it becomes a stray file.
10. Plain JavaScript by default; TypeScript only when explicitly asked — then also add
    `tsconfig.json` plus the matching `@types/*` packages.

## Recipes / workflows

**Create a playground from scratch.**

1. Choose the runtime (Choosing a runtime) — default to static for anything that is fundamentally
   an HTML+JS page; vite when you need a framework or a build step.
2. Follow that runtime's rules: canonical `package.json` with `dev-port` respecting `$PORT`,
   Tailwind 4 wired, imports resolvable without a bundler (static) or declared in package.json
   (vite).
3. Read the matching example before emitting (Examples as fixtures) and match its shape.
4. Run the pre-output checklist below.

**Edit an existing playground.**

When given current files plus a change request:

1. Output changed/added files only (JSON) or the complete updated tree (zip) — see above.
2. For JSON output, include `/changes.md`: short plain-language markdown explaining what you changed and why, for the person reading your answer ("Added a score counter that updates when…"). It is stripped from the sandbox and rendered as your explanation — if it's missing, your answer renders empty. Never put `changes.md` in a zip: importers have no renderer there and it becomes a stray file.

## Examples as fixtures

Two complete worked outputs ship with this skill — one per runtime. Each is a task line plus the exact JSON an agent should emit:

- `examples/static-p5.md` — static p5.js bouncing-ball exercise (CDN ESM import, canonical package.json).
- `examples/vite-react.md` — lean Vite + React tabs demo with Tailwind 4.

Each is verified by a gate (`npm run verify:lms-playground-skill`) that parses its fenced map the way the platform does and asserts the output contracts (bare maps, canonical/lean package.json, Tailwind 4 wiring) — they must stay copy-paste-able as real playground file sets.

Match their shape: file naming sensibility, comment density, dependency discipline, and key order (primary editable file first). Read the example that matches what you are about to build before emitting output.

## Pre-output checklist

1. Runtime chosen for the task (static when CDN-sufficient)?
2. `package.json`: correct canonical scripts + `dev-port` with `$PORT`; lean or CDN-only deps as appropriate?
3. Tailwind 4 wired for the runtime in use?
4. Imports resolvable without a bundler (static) or declared in package.json (vite); JS unless TS was requested?
5. Key/file order: primary student-editable file first; zip flat-rooted with no dotfiles/lockfiles/node_modules?
6. Edit request → partial JSON map + `/changes.md` (or full tree if deleting files); never `changes.md` in a zip?
7. Output is exactly one bare JSON map — or exactly one zip — nothing else?

## Cross-reference map

- **codehike-content** — owns MDX mechanics, annotations, step layouts, prose style. Hand off when
  the deliverable is raw CodeHike MDX for an element's content field; a code-playground's
  `instruction` and `hint` fields are CodeHike-routed and delegate to it.
- **lms-elements** — owns the 11 authorable element types' `data` shapes and minimal records. Hand
  off when the deliverable is a whole element record `{type, data}`; its code-playground section
  delegates file-set authoring back here and documents the `files` wrapper your map lands in
  (`code-playground.data.files.files`).
- **lms-backup** — owns whole backup JSON files: envelope, id/rank fabrication schemes, embedding
  records into files, the validation loop. It embeds element records (with your file set inside)
  verbatim; JSON escaping is its job.
