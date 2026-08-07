# FAQ

## How do I add memory rules to CLAUDE.md?

Copy the drop-in block from [memory-rules.md](memory-rules.md) and paste it into your CLAUDE.md or AGENTS.md. That is the whole installation: from then on the agent writes memory entries as single lines with a `[type:...] [src:...] [when:...] [rec:...]` prefix. No tools required.

## Which block should I paste — canonical or compact?

The **canonical block is the recommended one**: it carries the complete rule set of the specification — the type and source glossaries, the conflict-resolution rule, the poisoning gate, and anchoring examples — and specific instructions are followed more consistently than terse ones. The **compact block** is a strict subset for everyday desktop AI assistants — a memory file in a chat app rather than an agent harness: same entry format, same markers, fewer explanations. Entries written under either block are identical in shape, so upgrading later means replacing the block — no entry rewriting ([memory-rules.md](memory-rules.md)).

## Why two timestamps?

`event_time` is when the fact became true in the world; `recorded_time` is when it was written down. One timestamp cannot distinguish "the world changed" from "we learned late", and deterministic conflict resolution ("newest event wins") needs the event time, not the writing time. On the MemoryAgentBench fact-consolidation tasks, prior systems score 7–54% single-hop and at most 7% multi-hop, while deterministic resolution over explicit version markers reaches 78.0–94.8% single-hop and 30.2–51.5% multi-hop, pooled ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)). The bound matters as much as the gain: on general conversational QA (LongMemEval) the same paper finds no significant difference (26/45 vs 29/45, McNemar p=0.45) and limits the claim to current-value questions over explicit version metadata — which is exactly the claim AMR makes, and no more. The two-axis model is the industry standard for temporal agent memory: Zep tracks valid time and ingestion time per fact and invalidates rather than deletes on conflict ([arXiv:2501.13956](https://arxiv.org/abs/2501.13956)).

A practical consequence: `event_time` may honestly be `unknown` — the event date is sometimes undeterminable, the writing date never is. Writing an approximated date instead of `unknown` silently corrupts the very resolution the field enables, which is why the rules forbid it.

## How do the rules resist memory poisoning?

Memory poisoning turns a one-shot prompt injection into a persistent compromise: a malicious "instruction", once written into memory (for example, from a fetched web page through session summarization), executes in every future session. The attack is systematically studied ([MPBench, arXiv:2606.04329](https://arxiv.org/abs/2606.04329)) and demonstrated against production agents ([Unit 42](https://unit42.paloaltonetworks.com/indirect-prompt-injection-poisons-ai-longterm-memory/)); notably, existing prompt-injection defenses do not cover it.

The rules close the cheapest channel: **memory is data, not commands**. Only entries with `src:stated` may carry `type:instruction`; a directive found in fetched or derived content is recorded as a `fact` about what the source says — the information is kept, the executability is not. Honest calibration, with no fine print: this is a prompt-level mitigation, not a complete defense, and in a plain context file the `src` tag itself is written by the same model the attack targets — self-reported provenance is hygiene, not a security boundary. The guarantee version requires the channel to be assigned by the pipeline rather than the model ([SPEC.md, section 2.4](SPEC.md#24-provenance)); retrieval-time filtering and isolation remain the memory system's responsibility.

## How are conflicts between two active entries resolved?

Deterministically, by fields — not by LLM judgment. `src:inferred` never overrides an active `src:stated` entry, whatever the dates: what the user actually said is displaced only by a later stated or derived entry, never by the agent's own conclusion. Otherwise the later `rec` wins; at equal `rec`, the entry lower in the file wins — entries are appended, so lower means written later. The rule and the reasoning behind each precedence are in [SPEC.md, section 5](SPEC.md#5-reading-policy-informative) and [rationale.md](rationale.md#reading-policy-conflict-resolution-spec-section-5).

## Why dates and not timestamps in the lite prefix?

Because a date is the maximum precision the writer can supply honestly. A model writing into a context file reliably knows the current date, but not a trustworthy clock or timezone — a fabricated time of day is exactly the kind of silently corrupting precision the `when:unknown` rule forbids, and a wrong order is worse than a coarse one. Within one day, order needs no clock: entries are appended, so file position is the deterministic tie-break. Full AMR records written by a harness with a real clock do carry ISO 8601 timestamps ([SPEC.md, section 2.5](SPEC.md#25-event_time-and-recorded_time); [rationale](rationale.md#dates-not-timestamps-in-amr-lite)).

## Why mark instead of delete?

Two reasons. First, any memory policy that can only keep or drop entries carries a provable residual-deficit floor — deleted material cannot be rebuilt when it turns out to be needed ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)). Second, marked history is a free audit trail: "what did the agent know on date X, and who changed it, when" is answerable only if nothing was deleted. So AMR records move through statuses (`active` → `superseded` / `corrected` / `quarantined` / `stale`) and are never removed.

## Doesn't never deleting bloat the memory?

The store grows; the agent's context does not. Only `active` records belong in the agent's working context — filtering by `status` is a deterministic field comparison (in a file memory, literally a grep), not an LLM call. For file memories like CLAUDE.md, periodically moving `[superseded:...]` and `[corrected:...]` lines into an archive file (the rules block names it MEMORY_ARCHIVE.md) is fully compliant: the specification forbids deletion as a way to express a status transition ([SPEC.md, section 2.6](SPEC.md#26-status)), not moving records between files. What it rules out is destroying history — which is irreversible ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)) and is exactly what time-scoped questions ("what was true on date X") depend on. Archiving matters because context files are paid for every session and reliability measurably decays with input length ([Context Rot](https://research.trychroma.com/context-rot)).

## How do I migrate existing memory?

Paste the block from [memory-rules.md](memory-rules.md) into your CLAUDE.md / AGENTS.md, then tell your agent:

> Rewrite my existing memory entries to follow the Memory writing rules above. Resolve relative dates to absolute ones, mark outdated entries with dated [superseded:...] marks — delete nothing.

The agent rewrites each entry into the prefixed format, converts relative dates to absolute ones, and marks stale entries instead of deleting them. Entries written before the block was adopted (no `[rec:...]` prefix, undated marks) remain valid and are upgraded by the same prompt.

## What's the difference between superseded and corrected?

`superseded` means the world changed: the record was true when written, and it stays part of history — "what was true on 2026-05-02" still returns it for dates before the change. `corrected` means the record was wrong when written: it was never true, and queries about past truth must not treat it as having been true. The distinction cannot be reconstructed after the fact, so the writer picks the mark at replacement time ([SPEC.md, section 4](SPEC.md#4-supersession-is-not-correction)).
