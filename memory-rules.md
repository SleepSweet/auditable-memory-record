# Memory writing rules

Memory writing rules are a drop-in block for an agent context file (CLAUDE.md, AGENTS.md, or equivalent) that teaches the agent to write memory entries in a single-line, metadata-prefixed format. Entries written this way follow the [AMR-lite](amr-lite.md) profile of the [Auditable Memory Record](SPEC.md) specification.

Adoption is copy-paste plus one sentence to your agent. No tools required.

## Minimal block

```
## Memory writing rules
Write every memory entry as one line:
`[type:{fact|instruction|preference|decision|gotcha}] [src:{stated|derived|fetched|inferred}] [when:YYYY-MM-DD] <content>`
`when` = the date the fact became true, absolute only — never "yesterday" or "last week".
Never overwrite or delete an entry: add a new one and mark the old by appending `[superseded]` (world changed) or `[corrected]` (was recorded wrong).
Never write secrets, tokens, or keys into memory.
<!-- memory rules v1 — github.com/sleepsweet/auditable-memory-record -->
```

## Full block

```
## Memory writing rules
When adding any entry to memory (CLAUDE.md, AGENTS.md, or auto-memory),
format it as a single line with a metadata prefix:

`[type:{fact|instruction|preference|decision|gotcha}] [src:{stated|derived|fetched|inferred}] [when:YYYY-MM-DD] <content>`

- `src`: stated = user said it; derived = verified from code/files;
  fetched = found externally; inferred = my own conclusion.
- `when` = the date the fact became true (not today's date), absolute only —
  never "yesterday" or "last week".
- Never overwrite an existing entry to change a fact. Add a new entry and
  mark the old one by appending `[superseded]` (world changed) or
  `[corrected]` (was recorded wrong). Do not delete history.
- Never write secrets, tokens, or keys into memory.
<!-- memory rules v1 — github.com/sleepsweet/auditable-memory-record -->
```

## Migrating existing memory

Paste the block into your CLAUDE.md / AGENTS.md, then tell your agent:

> Rewrite my existing memory entries to follow the Memory writing rules above. Mark outdated entries as [superseded] — delete nothing.

## What to expect

Honest calibration, so you can decide whether this is worth five lines of your context file:

- Agents follow the format roughly 70–90% of the time. Occasional unprefixed entries still happen; the migration prompt above also fixes those retroactively.
- The immediate effect on task success is modest — single-digit percentages — plus the elimination of specific failure classes: stale relative dates ("yesterday" written six weeks ago) and resurrected superseded decisions.
- The main value is downstream: memory that carries type, source, and date per entry stays cheap to maintain, deduplicate, and consolidate — by any tool, or by the agent itself.

The research behind each rule is in [rationale.md](rationale.md).
