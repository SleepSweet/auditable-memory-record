# FAQ

## How do I add memory rules to CLAUDE.md?

Copy the drop-in block from [memory-rules.md](memory-rules.md) — minimal (5 lines) or full — and paste it into your CLAUDE.md or AGENTS.md. That is the whole installation: from then on the agent writes memory entries as single lines with a `[type:...] [src:...] [when:YYYY-MM-DD]` prefix. No tools required.

## Why two timestamps?

`event_time` is when the fact became true in the world; `recorded_time` is when it was written down. One timestamp cannot distinguish "the world changed" from "we learned late", and deterministic conflict resolution ("newest event wins") needs the event time, not the writing time. On the MemoryAgentBench fact-consolidation tasks, systems without deterministic resolution over version markers score 7–54%, while deterministic resolution reaches 78.0–94.8% ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)).

## Why mark instead of delete?

Two reasons. First, any memory policy that can only keep or drop entries carries a provable residual-deficit floor — deleted material cannot be rebuilt when it turns out to be needed ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)). Second, marked history is a free audit trail: "what did the agent know on date X, and who changed it, when" is answerable only if nothing was deleted. So AMR records move through statuses (`active` → `superseded` / `corrected` / `quarantined` / `stale`) and are never removed.

## How do I migrate existing memory?

Paste the block from [memory-rules.md](memory-rules.md) into your CLAUDE.md / AGENTS.md, then tell your agent:

> Rewrite my existing memory entries to follow the Memory writing rules above. Mark outdated entries as [superseded] — delete nothing.

The agent rewrites each entry into the prefixed format, converts relative dates to absolute ones, and marks stale entries instead of deleting them.

## What's the difference between superseded and corrected?

`superseded` means the world changed: the record was true when written, and it stays part of history — "what was true on 2026-05-02" still returns it for dates before the change. `corrected` means the record was wrong when written: it was never true, and queries about past truth must not treat it as having been true. The distinction cannot be reconstructed after the fact, so the writer picks the mark at replacement time ([SPEC.md, section 4](SPEC.md#4-supersession-is-not-correction)).
