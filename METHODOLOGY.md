# Methodology: from project gotchas to a durable agent wiki

Capturing lessons per project (Tier 1) is the easy, high-value habit. This
document is the harder half: how to integrate lessons **across** projects into a
durable knowledge base that an LLM can load, trust, and not be poisoned by.

The capture file is a notebook. The wiki is a reference the agent reads back into
its context. They are different artifacts with different rules — the most common
mistake is treating them as the same file.

## 1. Two tiers, opposite biases

| | Capture buffer (`thoughts/lessons.md`) | Durable wiki |
|---|---|---|
| Scope | one project | all projects |
| Optimize for | **recall** — write it down, sort later | **precision** — every entry costs context budget |
| Tone | raw, specific, timestamped | synthesized, general, evergreen |
| Failure mode | a real lesson never written down | a noisy wiki that's expensive to load and easy to distrust |

Write captures freely; the buffer is allowed to be messy. Promote
conservatively; the wiki is not.

**One buffer, many agents.** Both tiers are plain, tool-neutral files, which is
what lets a project's agents *share* them. If you orchestrate with one agent and
execute with another (a common split — one plans, another implements), point
both at the same `thoughts/lessons.md`: a gotcha the executor hits during
implementation is then waiting for the orchestrator next session, and vice
versa. The neutral file is the shared memory between heterogeneous agents on the
same work — and the durable wiki, likewise tool-neutral, is read back by
whichever agent is running. Recurrence *across different agents on the same
project* is a weaker promotion signal than across projects (they share a
codebase), but a lesson two different agents independently trip over is still
worth a hard look. One caveat for a shared buffer: it needs append-discipline —
two agents writing concurrently can clobber each other, so append entries rather
than rewriting the file.

**Never write straight to the wiki.** A lesson learned once, in one project, is
a hypothesis. Putting hypotheses in the durable store is how it fills with
project quirks that don't generalize.

## 2. The promotion gate: recurrence, not importance

A lesson is promoted when it **recurs in a second, independent context** — not
when it feels important.

Recurrence is the cheapest reliable signal that a lesson is a *general pattern*
rather than a property of one codebase. Importance is a feeling, and feelings
promote everything. Two independent occurrences is a good default threshold:
strict enough to filter one-offs, loose enough to catch real patterns early.

"Independent" matters: the same bug hit twice in one project is one occurrence.
Two *different* projects, or two genuinely different subsystems, is two.

Be strict about *independent*, though: a second hit in a *similar* domain often
illustrates the same pattern rather than proving it generalizes. When the
evidence is still a single domain, promote the entry as **`[provisional]`** —
kept in the wiki but tagged not-yet-confirmed-general — and upgrade it to durable
only when a genuinely cross-domain occurrence lands.

## 3. Synthesize, don't copy

When a lesson clears the gate, do **not** paste the project entry into the wiki.
Abstract it:

- Strip the project-specific surface (file names, your local paths, the exact
  ticket).
- Keep the transferable principle — the thing that will be true in the next
  project too.
- Cite the occurrences as *evidence*, not as the content. The project entries are
  the witnesses; the wiki entry is the generalization they support.

A good wiki entry reads like a rule with a reason, followed by "seen in: A, B."
A bad one reads like a copied diary entry.

**Some lessons are procedures — promote them to skills, not prose.** Not every
durable lesson is a fact. A fact ("the pool's acquire timeout defaults to
unbounded") belongs in the wiki as an entry the agent re-reads. A *repeatable
check or procedure* is more reliable as something that **fires** than as text the
model must remember and choose to apply — so promote it to a named skill, a lint
rule, or a pre-commit/CI hook instead. The worked example's pool-timeout lesson
(§8) is really a check — grep the config for unbounded pool acquire timeouts —
not a paragraph to recall.

## 4. Write for an LLM reader, not a human archivist

The durable wiki's whole purpose is to be loaded back into an agent's context.
That changes how entries should be written:

- **Lead with the trigger.** A one-line "when does this apply" up front, so a
  recall/search step can match it before the agent has spent budget reading the
  body. ("When calling an SDK permission callback…", not a title like "Permissions".)
- **Make each entry self-contained.** It will be retrieved in isolation, without
  the entries around it. No "as noted above."
- **Tag durability and version-sensitivity.** LLM-consumed facts go stale, and a
  stale fact recalled *confidently* is worse than no fact. Mark "true as of
  vX.Y" so a future reader knows to re-verify. Version-pinned details and durable
  principles deserve different trust.
- **Link related entries.** A traversable graph beats a flat list; one recalled
  entry should point at its neighbors.
- **Stay small and high-precision.** Every entry is a tax on every future
  context load. When in doubt, don't promote — or merge into an existing entry
  rather than adding a new one.

## 5. The integration pass (the "ingest flow")

Promotion is a periodic pass, run on a cadence or triggered when a capture buffer
flags a recurrence:

1. **Scan** the capture buffers across projects.
2. **Cluster** entries by theme.
3. **Detect recurrence** — clusters with ≥2 independent occurrences.
4. **Draft or update** a generalized wiki entry per surviving cluster
   (synthesize, §3; write for recall, §4).
5. **Link back** to the source occurrences as evidence.

Cross-project recurrence is only detectable if something can *see* across
projects. Per-project files can't see each other, so the pass needs either a
place where candidates are aggregated, or a step that reads each project's buffer
in turn. That aggregation point — not the capture habit — is the real
infrastructure decision.

## 6. Decay and verification

Durable is not immortal. Three rules keep the wiki from rotting:

- **Re-verify on use.** Before acting on a recalled lesson, confirm it still
  holds — especially version-pinned ones. The strongest meta-lesson most agents
  learn is that *runtime beats memory and types*: a confidently recalled fact is
  still a claim until the live path agrees. The wiki should carry that posture
  about its own entries.
- **Prune on contradiction.** When a recalled lesson turns out wrong, fix or
  delete it in the same pass. A wrong entry that survives is worse than a missing
  one, because it's trusted.
- **Prune dead weight.** Judge the wiki by *contribution rate* — how often an
  entry actually fires at point of use — not by entry count. A well-written
  lesson nobody ever recalls is dead weight; sweep periodically and cut entries
  that never fire, alongside the contradiction prune above.

## 7. The read side: closing the loop

Capture and promotion are write paths. None of it prevents a repeat unless the
stored lesson is *read back* — and the two tiers are read differently, which is
what keeps recall cheap.

- **Capture buffer → read whole, once.** A project's `lessons.md` is small and
  scoped, so the cheapest correct move is to read it at the start of a session in
  that repo. It's a one-time cost — a few hundred to a couple thousand tokens,
  cached for the session — not per-turn latency. Keep it that way by keeping the
  file bounded: the dedupe rule plus promotion (which moves durable entries out
  to the wiki) stop it from ballooning.
- **Durable wiki → retrieve on demand.** The wiki aggregates every project and
  grows unbounded; loading all of it would be the token cost people fear. So you
  don't load it — you query it for the few entries relevant to the task at hand.
  This is the whole reason §4 insists entries be trigger-first and retrievable by
  description: it makes just-in-time lookup accurate and cheap.

Mechanism matters for reliability. A passive "read it at session start"
instruction is the tool-neutral baseline, but agents are inconsistent about
honoring "always do X first." Where it matters, wire the read through the tool
itself — a session-start hook that injects the file removes any reliance on the
model remembering. The instruction is the portable floor; a hook is the
guarantee.

## 8. Worked example

One gotcha, traced end to end: two project captures, one promoted wiki entry.

**Capture in project A** (`billing-api`, a Postgres-backed service). Raw, in
that project's `thoughts/lessons.md`:

> ## 2026-02-03 — Pool acquire hangs forever under load
> Context: requests piled up and timed out during a traffic spike; the DB itself
> was healthy.
> - The connection pool's `acquireTimeout` defaults to *unbounded*. When a
>   handler leaked a connection, later callers blocked on checkout forever
>   instead of failing fast — so one leak cascaded into a full stall.
> - The query timeout we'd set didn't help: it bounds a query *once you hold a
>   connection*, not the wait to acquire one. Different timeout, different knob.
> - Fix: set an explicit `acquireTimeout` so checkout fails fast and surfaces the
>   real leak.
> Durability: likely-general — every pool I've used has this shape.

A single occurrence. It's a hypothesis, not wiki-worthy yet (§2).

**Recurrence in project B** (`search-indexer`, unrelated service, a Redis client
pool — different language, different dependency). Months later, its own
`thoughts/lessons.md`:

> ## 2026-05-19 — Redis pool checkout blocks instead of erroring
> Context: indexer went unresponsive after a burst; no errors, just stalled.
> - Pool checkout has no timeout by default; a saturated pool turned into hung
>   callers, not exceptions. Same shape as the billing-api pool hang.
> - Fix: explicit pool-checkout timeout, separate from the per-command timeout.
> Durability: likely-general — promotion candidate, recurs across projects.

Two independent occurrences, two different stacks (§2). It clears the gate.

**Promoted wiki entry.** Synthesized, not copied (§3); written trigger-first and
durability-tagged for an LLM reader (§4):

> **Trigger:** configuring any client-side resource pool — DB connections, HTTP
> keep-alive, Redis, thread pools.
>
> A pool's *acquire/checkout* timeout is commonly **unbounded by default**, and
> it is a different knob from the per-operation timeout. With no acquire timeout,
> a leaked or saturated pool degrades into callers hung indefinitely on checkout
> instead of fast, visible failures — converting a local leak into a full stall.
> Always set an explicit acquire timeout, distinct from the operation timeout, so
> exhaustion fails fast and points at the real cause.
>
> Durability: durable principle, not version-pinned — verify the parameter name
> per library, but the default-unbounded shape recurs across pool
> implementations. Related: [[operation-vs-connection-timeouts]],
> [[fail-fast-on-resource-exhaustion]].
>
> Seen in: `billing-api` (Postgres pool, 2026-02), `search-indexer` (Redis pool,
> 2026-05).

Note what the promotion did: dropped the file names and the specific libraries,
kept the transferable principle, led with a matchable trigger, tagged what's
durable versus what to re-verify, and cited the two captures as *evidence* rather
than pasting them. The diary entries are the witnesses; the wiki entry is the
generalization they support.

Per §3, because this particular lesson is a *check*, its strongest promotion is
also a lint rule that greps for unbounded acquire timeouts; the prose entry above
illustrates the synthesize-don't-copy and write-for-recall mechanics, which apply
whichever form the promotion takes.

## Reference note

This is a method, not a tool. One instantiation is a personal LLM-native
knowledge base with an ingest flow that performs the §5 pass; another is a
plain markdown wiki you synthesize by hand on a monthly cadence. The tiers, the
recurrence gate, and the LLM-reader writing rules are what carry across — the
storage backend is interchangeable.
