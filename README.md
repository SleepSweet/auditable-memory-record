# Auditable Memory Record (AMR)

The Auditable Memory Record (AMR) is an open specification defining the fields every AI agent memory record should carry — type, content, content hash, provenance channel, event time and recorded time, status, and redaction marker — together with the published research rationale for each field. A memory system that stores these fields is AMR-compliant.

[Specification](SPEC.md) · [Memory rules (drop-in block)](memory-rules.md) · [AMR-lite profile](amr-lite.md) · [Rationale](rationale.md) · [FAQ](faq.md) · [JSON Schema](schema/amr.schema.json) · [Examples](examples/)

Status: **v0.1-draft**, open for comment through [issues](https://github.com/sleepsweet/auditable-memory-record/issues).

## Start in 30 seconds

Paste this into your CLAUDE.md / AGENTS.md:

```
## Memory writing rules
Write every memory entry as one line:
`[type:{fact|instruction|preference|decision|gotcha}] [src:{stated|derived|fetched|inferred}] [when:YYYY-MM-DD] <content>`
`when` = the date the fact became true, absolute only — never "yesterday" or "last week".
Never overwrite or delete an entry: add a new one and mark the old by appending `[superseded]` (world changed) or `[corrected]` (was recorded wrong).
Never write secrets, tokens, or keys into memory.
<!-- memory rules v1 — github.com/sleepsweet/auditable-memory-record -->
```

Then, to migrate what is already there, tell your agent:

> Rewrite my existing memory entries to follow the Memory writing rules above. Mark outdated entries as [superseded] — delete nothing.

That is the whole adoption path. A longer block with explanations, and an honest calibration of what this does and does not buy you, is in [memory-rules.md](memory-rules.md).

## Why these fields

Three published results carry most of the specification. 97.7% of unverifiability errors in agent memory are associated with write-time contamination, and type-aware memory improves reliability by up to 28.27% while retrieving up to 5.8x fewer tokens ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) — hence `type` and `provenance` at write time. On MemoryAgentBench fact consolidation, existing memory systems resolve conflicting facts at 7–54% (Zep 7%, Mem0 18% single-hop), while deterministic resolution over explicit version markers reaches 78.0–94.8% ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)) — hence `event_time`, `recorded_time`, and supersession links. And any keep-or-delete forgetting policy carries a provable residual-deficit floor ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)) — hence a `status` ladder instead of deletion. The full field-by-field grounding is in [rationale.md](rationale.md).

## Conformance levels

- **AMR-Core** — the record carries all MUST fields of the [specification](SPEC.md).
- **AMR-Extended** — all MUST and all SHOULD fields.
- **AMR-lite** — the human-readable profile for plain-text file memories: the author writes `[type] [src] [when]` prefixed lines, a normalizer derives the rest. AMR-lite + normalizer = AMR-Core ([amr-lite.md](amr-lite.md)).

Records validate against [schema/amr.schema.json](schema/amr.schema.json) (JSON Schema draft 2020-12).

## Show that you follow it

If your project's memory follows the rules, add the badge to your README — it links readers to the specification and makes adoption findable:

[![memory: AMR-lite](https://img.shields.io/badge/memory-AMR--lite-blue)](https://github.com/sleepsweet/auditable-memory-record)

```markdown
[![memory: AMR-lite](https://img.shields.io/badge/memory-AMR--lite-blue)](https://github.com/sleepsweet/auditable-memory-record)
```

## Implementations

- [sleepsweet](https://github.com/sleepsweet/sleepsweet) — reference implementation: an engine + CLI that consumes AMR records and acts as an AMR-lite normalizer.
- *Your implementation here* — open a PR adding one line to this list; any tool that reads or writes AMR records qualifies.

## Related work

- [AGENTS.md](https://agents.md) — the sibling convention for agent *instructions*. AMR standardizes agent *memory records*; the two compose — an AGENTS.md file tells the agent how to behave, AMR defines the shape of what it writes down.
- [memory-hygiene-bench](https://github.com/sleepsweet/memory-hygiene-bench) — open benchmark measuring the cost of degraded agent memory; an intervention benchmark measuring the effect of these rules is being developed there.

## Contributing

Changes go through an RFC-lite process: an issue with a stated rationale — a published research result or a production case. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

CC BY 4.0
