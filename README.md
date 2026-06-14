# agent-lessons-loop

A lightweight practice for turning the gotchas your coding agent hits into
durable, cross-project knowledge — instead of rediscovering the same snags
every session.

**Who's this for:** anyone running one or more coding agents who keeps
re-hitting the same non-obvious snags and wants them captured once and reused.

A coding agent — Claude Code, Codex, Cursor, whatever you run — keeps
relearning the same non-obvious things: the type that lies, the endpoint that
429s, the flag that's required-despite-being-optional. The fix is cheap: one
passive instruction plus a place to write things down, and a periodic pass that
lifts what recurs into a knowledge base the agent actually reads. This repo is
that practice, packaged to drop into any project.

**Platform-agnostic by design.** The instruction references only a plain
`thoughts/lessons.md`, so it works in any agent — only the install location
differs per tool (see the [placement table](./global-instruction-snippet.md)).
The payoff is sharpest when you run more than one: an orchestrator and an
executor agent (e.g. Claude planning, Codex executing) **share one capture
buffer**, so a lesson either one learns is there for the other next session.

## Three tiers

| Tier | Where | Cost | Bias |
|------|-------|------|------|
| **1. Capture** | per-project `thoughts/lessons.md` | ~free, passive | high recall, noisy by design |
| **2. Promote** | a periodic synthesis pass | minutes, occasional | recurrence-gated |
| **3. Recall** | a durable, agent-readable wiki | loaded into context | high precision, small |

Capture is nearly free and gives most of the value on its own. Promotion is what
makes it *compound across projects*. Recall is the payoff — a small wiki the
agent loads and trusts. The durable form needn't be prose: a lesson that's a
repeatable procedure is better promoted to a skill or check — a lint rule or hook
that *fires* — than to an entry the agent has to remember to apply.

## Quick start — capture only (2 minutes)

**Let an agent do it (recommended).** Link or clone this repo and tell your
coding agent *"install this for me."* It follows
[`INSTALL.md`](./INSTALL.md): picks the right destination for your tool, appends
the habit once (idempotent — it won't double-install), and confirms.

**Or by hand.** Append
[`install/lessons-instruction.md`](./install/lessons-instruction.md) to your
agent's always-on instructions — `AGENTS.md` at the repo root for the cross-tool
default, or your agent's global file so it applies to every project:

```sh
# Claude Code, global (all projects):
cat install/lessons-instruction.md >> ~/.claude/CLAUDE.md
```

The [install guide](./global-instruction-snippet.md) has the placement table and
the command for each tool. (`>>` appends — it won't clobber an existing
instruction file.) That's the whole loop — re-read this project's lessons on
session start, capture new ones on a snag. Seed new projects with
[`templates/lessons.md`](./templates/lessons.md) if you want a header, or just
let the agent create the file on the first snag.

## The full loop

Capture alone is a per-project notebook. The leverage is **cross-project**: a
gotcha that bites you in two unrelated repos is almost certainly a general truth
worth keeping. [`METHODOLOGY.md`](./METHODOLOGY.md) covers the hard part — how to
promote recurring lessons into a durable wiki an LLM can actually use without it
rotting or bloating your context budget. For one gotcha traced end to end — two
project captures promoting to a single generalized wiki entry — see the
[worked example](./METHODOLOGY.md#8-worked-example).

## Why it works

- **Capture is passive, not a tool you remember to call.** It rides in the
  agent's standing instructions, so it happens during normal work.
- **The promotion gate is recurrence, not importance.** Recurrence across
  independent projects is the signal that something generalizes; importance is a
  feeling that fills wikis with one-off trivia.
- **The two tiers have opposite biases on purpose.** The capture buffer
  optimizes recall (write it down, sort later). The durable wiki optimizes
  precision (every entry costs tokens when loaded, so noise is expensive).

## Status

Methodology + drop-in instruction. Tool-agnostic: the durable wiki can be a
markdown folder, a notes app, or an LLM-native knowledge base with an ingest
flow — the method doesn't care which.

## License

[MIT](./LICENSE).
