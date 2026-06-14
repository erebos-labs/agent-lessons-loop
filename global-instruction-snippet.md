# Drop-in instruction (any agent)

The instruction is tool-neutral — it only references a plain `thoughts/lessons.md`
file, which any coding agent can append to and read back. **Only the location
changes per tool; the text is identical everywhere.**

The instruction text lives in
[`install/lessons-instruction.md`](./install/lessons-instruction.md) — one clean
block you **append** to your tool's always-on file (don't overwrite an existing
one).

## Where to put it

Two ways, depending on whether your tool has an always-on *global* file:

- **Global (passive, all projects).** Tools with a user-level instruction file
  load it into every session automatically — append it once and every project
  inherits it. Best for your primary agent.
- **Project (one file, many tools).** `AGENTS.md` at the repo root is the
  closest thing to a cross-tool standard and is read by a growing set of agents
  (Codex among them). Append it there once per repo and every tool that honors
  `AGENTS.md` picks it up — which is also what lets an orchestrator and an
  executor agent share the same lessons file.

| Agent / tool | Always-on file to append to |
|---|---|
| **AGENTS.md standard** (Codex CLI & a growing set) | `AGENTS.md` at repo root *(recommended neutral home)* |
| **Claude Code** | `~/.claude/CLAUDE.md` (global, all projects) or `./CLAUDE.md` (project) |
| **Cursor** | `.cursor/rules/lessons.mdc` (or legacy `.cursorrules`) |
| **Anything else** | wherever instructions are always loaded into the system prompt |

> Instruction-file conventions vary by tool and change between versions — confirm
> your tool's current path. Start with `AGENTS.md`; add your primary agent's
> native global file for passive, all-projects behavior.

## Install

Append the block to the destination from the table above. For example:

```sh
# Cross-tool default (run at a repo root):
cat install/lessons-instruction.md >> AGENTS.md

# Claude Code, global (all projects):
cat install/lessons-instruction.md >> ~/.claude/CLAUDE.md

# Cursor (run at a repo root):
cat install/lessons-instruction.md >> .cursor/rules/lessons.mdc
```

`>>` appends, so an existing instruction file is preserved. The leading HTML
comment in the block is a marker you can search for to update or remove it later.
