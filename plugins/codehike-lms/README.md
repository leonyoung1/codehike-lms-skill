# CodeHike LMS

A Codex plugin by Leon Young for writing student-facing LMS2 lessons as CodeHike MDX. It bundles the `codehike-content` skill, its annotation reference, and two example documents.

Use it for annotated code, Scrollycoding walkthroughs, Spotlight comparisons, code tabs, Mermaid diagrams, alerts, accordions. The output is raw MDX that you can paste into an LMS2 lesson element's `code` field.

## Install

This repository doubles as its own [plugin marketplace](https://developers.openai.com/plugins/build/plugins). Add the source once:

```bash
codex plugin marketplace add leonyoung1/codehike-lms-skill
```

Then open Codex in the ChatGPT desktop app → **Plugins** tab → pick the **CodeHike Plugins** source → install **codehike-lms**. For team distribution, a workspace admin can import this repository instead: **Admin → Plugins → Import marketplace**, with Source `https://github.com/leonyoung1/codehike-lms-skill`.

Other routes: if you received a Codex plugin share link, open it and follow the installation flow. If you received the release ZIP:

1. Extract it and keep the entire `codehike-lms` folder, including the hidden `.codex-plugin` directory.
2. Open Codex and ask the built-in plugin creator to register the extracted plugin. Replace the path below with the actual folder path:

   ```text
   $plugin-creator Add the existing plugin at /path/to/codehike-lms to my personal marketplace, preserve its bundled files and metadata, and install it.
   ```

3. Start a new Codex task so the installed skill is available.

This is a skills-only plugin. It includes no MCP server, hooks, application connection, or API key requirement. Rendering the generated lessons requires LMS2's CodeHike components; the plugin does not install that renderer.

## Use

Select **CodeHike LMS** in the plugin picker, or invoke its bundled skill:

```text
$codehike-content Write an LMS2 lesson element explaining Python for loops to beginners. Include a small annotated example and a practice question.
```

Other requests to try:

- Create a three-step Scrollycoding walkthrough of JavaScript fetch and error handling.
- Compare JavaScript map and Python list comprehensions using CodeWithTabs.
- Write an LMS2 lesson explaining request validation with a small Mermaid diagram.

The skill returns the lesson content without a preamble or an outer code fence. Copy the full response into one LMS2 lesson element.

## Share

In Codex, open the plugin details and use **Share**, or send the complete release ZIP and these installation instructions. A local filesystem path only works on the computer where that path exists.

For public directory distribution, follow OpenAI's [plugin submission documentation](https://developers.openai.com/plugins/deploy/submission). This package has not been submitted to the public directory.

## Contents and compatibility

In this repository the plugin lives under `plugins/codehike-lms/`, exposed by `.agents/plugins/marketplace.json`. The package itself contains:

- `plugin.json` + `.codex-plugin/plugin.json`: plugin identity, version, and metadata (portable Agent Plugins manifest plus Codex compatibility overlay); OpenAI presentation settings live in the root manifest's `interface` block.
- `skills/codehike-content/SKILL.md`: lesson authoring instructions. Skills are discovered automatically from the fixed `skills/` directory — no manifest declaration needed. See [OpenAI's plugin packaging documentation](https://developers.openai.com/plugins/build/plugins).
- `skills/codehike-content/reference/annotations.md`: annotation reference.
- `skills/codehike-content/examples/`: annotated-code and interactive layout examples.
- `UPSTREAM.md`: source revision and packaging notes.

The upstream skill is maintained from a private LMS repository. Its live rendering behavior remains the authority; this package cannot independently verify that renderer.
