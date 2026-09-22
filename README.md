# codehike-lms-skill

An [Agent Skill](https://agentskills.io) for writing LMS2 lesson content as MDX rendered by
[CodeHike](https://codehike.org): syntax-highlighted and annotated code blocks, scroll/click step
layouts (Scrollycoding, Spotlight), tabs of code variants, mermaid diagrams, alerts, and accordions.

**Workflow:** install it in your own Codex, ask the agent to write a lesson element, then copy
the generated MDX straight into an LMS2 markdown element — it compiles as-is. Start at
[SKILL.md](SKILL.md); `reference/` holds the full annotation reference and `examples/` two runnable
demo documents.

## Install in Codex

Inside a Codex session:

```text
$skill-installer install https://github.com/leonyoung1/codehike-lms-skill
```

Or manually — clone this repo and place its contents as a folder named `codehike-content` (the
skill's declared name) in your personal skills directory:

```bash
git clone --depth 1 https://github.com/leonyoung1/codehike-lms-skill /tmp/ch-skill \
  && mkdir -p ~/.codex/skills/codehike-content \
  && cp -r /tmp/ch-skill/. ~/.codex/skills/codehike-content/
```

This skill is maintained from a private LMS repository; if it ever disagrees with the live app,
the app wins.
