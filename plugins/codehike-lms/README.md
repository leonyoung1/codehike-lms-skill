# CodeHike LMS

A Codex plugin by Leon Young for authoring complete student-facing LMS2 content. It bundles four skills that hand off to each other: `codehike-content` writes lesson sections and inline fields as CodeHike MDX; `lms-elements` authors complete records for all 11 element types; `lms-playground` builds runnable static or Vite playground file sets for the in-browser sandbox; `lms-backup` assembles whole course/unit/lesson/section backup files that validate clean against the platform's import validator.

Use it for annotated code, Scrollycoding walkthroughs, Spotlight comparisons, code tabs, Mermaid diagrams, alerts and accordions (`codehike-content`); quiz questions with hints and explanations plus image/video/link/data-table elements (`lms-elements`); p5.js/Phaser or React/Vite exercise sandboxes (`lms-playground`); and ready-to-import backup JSON for a whole unit, lesson, or section (`lms-backup`).

## Install

This repository doubles as its own [plugin marketplace](https://developers.openai.com/plugins/build/plugins). The usual route is the Codex app (ChatGPT desktop): open **Plugins**, choose **Add plugin marketplace**, and fill in the dialog with:

| Field | Value |
| --- | --- |
| **Source** | `leonyoung1/codehike-lms-skill` — GitHub shorthand; a full Git URL works too |
| **Git ref** | `main` — leave it as is |
| **Sparse paths** | leave blank: the whole repository is this small marketplace, there is nothing to trim |

Click **Add marketplace**. A source named **CodeHike Plugins** appears in the plugin directory; open it and install **codehike-lms**, then start a new task so its skill is available. If an older release was already installed and content looks stale after a new publish, remove the marketplace source (or restart the app) — installed copies are cached per version under `~/.codex/plugins/cache/`.

With Codex CLI instead of the desktop app:

```bash
codex plugin marketplace add leonyoung1/codehike-lms-skill
# manage it later: codex plugin marketplace list | upgrade [codehike] | remove codehike
```

For team distribution, a workspace admin can import this repository instead: **Admin → Plugins → Import marketplace**, with Source `https://github.com/leonyoung1/codehike-lms-skill`.

Other routes: if you received a Codex plugin share link, open it and follow the installation flow. If you received the release ZIP:

1. Extract it and keep the entire `codehike-lms` folder, including the hidden `.codex-plugin` directory.
2. Open Codex and ask the built-in plugin creator to register the extracted plugin. Replace the path below with the actual folder path:

   ```text
   $plugin-creator Add the existing plugin at /path/to/codehike-lms to my personal marketplace, preserve its bundled files and metadata, and install it.
   ```

3. Start a new Codex task so the installed skill is available.

This is a skills-only plugin. It includes no MCP server, hooks, application connection, or API key requirement. Rendering the generated lessons requires LMS2's CodeHike components; the plugin does not install that renderer.

## Use

Select **CodeHike LMS** in the plugin picker, or invoke a bundled skill by name:

```text
$codehike-content Write an LMS2 lesson element explaining Python for loops to beginners. Include a small annotated example and a practice question.
```

Other requests to try (each routes to the right skill):

- Create a three-step Scrollycoding walkthrough of JavaScript fetch and error handling. → `codehike-content`
- Build a static p5.js playground exercise where students make dots grow as they appear. → `lms-playground`
- Create a multiple-choice element record about Python loops, with hint and explanation. → `lms-elements`
- Make an LMS2 backup file: a new unit with two lessons, each containing a markdown section and a code playground. → `lms-backup`, delegating content to the other three

The skills return raw content without a preamble or an outer fence: MDX for a lesson element's `code` field, a bare `{type, data}` record, a bare SandpackFiles map (or flat-root zip), or a complete backup JSON file.

## Share

In Codex, open the plugin details and use **Share**, or send the complete release ZIP and these installation instructions. A local filesystem path only works on the computer where that path exists.

For public directory distribution, follow OpenAI's [plugin submission documentation](https://developers.openai.com/plugins/deploy/submission). This package has not been submitted to the public directory.

## Contents and compatibility

In this repository the plugin lives under `plugins/codehike-lms/`, exposed by `.agents/plugins/marketplace.json`. The package itself contains:

- `plugin.json` + `.codex-plugin/plugin.json`: plugin identity, version, and metadata (portable Agent Plugins manifest plus Codex compatibility overlay); OpenAI presentation settings live in the root manifest's `interface` block.
- `skills/codehike-content/`: lesson authoring instructions (`SKILL.md`), its annotation reference (`reference/annotations.md`), and worked MDX examples (`examples/`). Skills are discovered automatically from the fixed `skills/` directory — no manifest declaration needed. See [OpenAI's plugin packaging documentation](https://developers.openai.com/plugins/build/plugins).
- `skills/lms-elements/`: element record shapes for all 11 authorable types, with one gate-verified example record per type (`examples/*.json`).
- `skills/lms-playground/`: static and Vite playground file-set rules (canonical package.json, CDN-only imports, Tailwind 4), with one worked example per runtime (`examples/*.md`).
- `skills/lms-backup/`: backup JSON format — envelope, id/rank schemes, editing recipes, validation loop — with real UI exports plus a hand-crafted kitchen-sink unit file (`examples/*.json`).
- `UPSTREAM.md`: source revision and packaging notes.

The upstream skills are maintained from a private LMS repository. Their live rendering behavior remains the authority; this package cannot independently verify that renderer.
