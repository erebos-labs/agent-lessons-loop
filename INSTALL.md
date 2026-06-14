# Installing this with an agent

This repo is meant to be installed by a coding agent: link or clone it and tell
your agent **"install this for me."** The runbook below is written for that agent
to follow.

## Runbook (for the installing agent)

**Goal:** append the lessons habit to the user's always-on agent instructions —
once, without overwriting anything.

The instruction to install is the single markdown block in
[`install/lessons-instruction.md`](./install/lessons-instruction.md). It carries a
leading `agent-lessons-loop` marker comment — use that marker to stay idempotent.

1. **Pick the destination (append target — never overwrite).**
   - Default, *global / all projects*: your own user-level instruction file, if
     you have one (e.g. Claude Code → `~/.claude/CLAUDE.md`). The placement table
     in [`global-instruction-snippet.md`](./global-instruction-snippet.md) maps
     the path per tool.
   - *This project only*: `AGENTS.md` at the target repo root.
   - If you can't tell which the user wants, ask: global or this repo only?

2. **Check it isn't already installed.** Search the destination for the string
   `agent-lessons-loop`. If it's there, stop and report — do **not** append a
   second copy.

3. **Append the block.** From a local clone:

   ```sh
   cat install/lessons-instruction.md >> <destination>
   ```

   If you don't have the file locally, read its contents and append them. Use
   append (`>>`) — never overwrite the destination.

4. **Verify.** Confirm the `## Lessons` block now appears exactly once at the end
   of the destination and the preceding content is untouched.

5. **Report.** Tell the user which file you wrote, whether it's global or
   project-scoped, and that capture + recall are now active. Optionally seed
   `thoughts/lessons.md` from [`templates/lessons.md`](./templates/lessons.md).
