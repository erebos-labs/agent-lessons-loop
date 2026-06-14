<!-- agent-lessons-loop — append this whole file into your agent's always-on instructions. See the repo README for per-tool destinations. -->

## Lessons

**Passive habit — read past lessons so you don't repeat them, and write down new ones so the next session doesn't pay again.**

At the start of a session, read `./thoughts/lessons.md` if it exists.

When you hit a non-obvious snag and apply a workaround — a gotcha where the docs, the types, or your assumption were wrong — append a dated entry to `./thoughts/lessons.md` (create it if absent).

- Format: **date · context · the lesson · durability guess** (project-specific vs. likely-general).
- Skip the trivial; capture what you'd want the *next* session to know.
- Before adding, skim existing entries — sharpen a related one instead of duplicating.
- Never capture secrets, credentials, tokens, internal hostnames, or customer identifiers — sanitize before writing, and again before promotion.
- When a lesson recurs in a second, independent project, flag it as a promotion candidate — a wiki entry if it's a fact, or a skill/check (lint rule, hook) if it's a repeatable procedure.
