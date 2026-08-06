# Field-by-field rationale

Every requirement in the [AMR specification](SPEC.md) traces to a published result. This document states, per field: the requirement, the reason, and the source. All numbers below are quoted as published; each citation was checked against the paper before inclusion.

## Why a write-time standard at all

Context files are not automatically useful. An evaluation of repository-level context files (AGENTS.md and similar) found that providing them "does not generally improve task success rates, while increasing inference cost by over 20% on average" — while also finding that "instructions in the context files are well followed by coding agents" ([arXiv:2602.11988](https://arxiv.org/abs/2602.11988)). The lesson for memory files is direct: agents will follow what you write, and unhelpful content costs real money — so what goes into a memory entry at write time decides whether the file pays for itself.

Write time is also where the damage happens. In MemGuard's analysis, "unverifiability errors are predominantly associated with write-time contamination (97.7%)" ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) — records that cannot be checked later are almost always born broken, not broken later. A standard that only governs retrieval or consolidation arrives too late; AMR therefore standardizes the record at the moment it is written.

## `type` (MUST)

**Requirement:** every record declares exactly one type.

**Reason:** different types age differently (a `fact` can go stale; a `preference` rarely does), compare differently, and must not be used as interchangeable evidence.

**Source:** MemGuard identifies "heterogeneous memory contamination" — functionally distinct memories collapsed into a shared space and misused as interchangeable evidence — and fixes it by assigning "each memory an explicit functional role at write time". The type-aware framework "improves memory reliability by up to 28.27% while retrieving up to 5.8x fewer memory tokens than prior methods" ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)).

## `content_hash` (MUST)

**Requirement:** SHA-256 of the record content, stored with the record.

**Reason:** exact deduplication, incremental re-processing, and tamper evidence all reduce to comparing hashes. A store reconciling two copies of a memory file can prove "same record" or "changed record" without an LLM call.

**Source:** the write-time contamination result above ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) motivates capturing integrity evidence at the single point where the record is created; hashing is the standard mechanism for it.

## `provenance.channel` (MUST), `source_ref` / `agent` / `model` (SHOULD)

**Requirement:** every record states how it was obtained (`stated` / `derived` / `fetched` / `inferred`); when known, also from what source and by which agent and model.

**Reason:** "the user said it" and "the agent concluded it" deserve different trust, different staleness policies, and different treatment under suspected poisoning. Without the channel recorded at write time, this distinction is unrecoverable later.

**Source:** 97.7% of unverifiability errors are associated with write-time contamination ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) — verifiability must be captured when the record is written. An append-only, provenance-carrying event log as the basis for auditable agent memory is demonstrated by projectmem, whose "immutable log also serves as a provenance trail for reproducible, auditable AI-assisted development" ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)).

## `event_time` + `recorded_time` (MUST)

**Requirement:** two timestamps per record — when the fact became true in the world, and when it was written down.

**Reason:** conflict resolution between contradictory facts can be done deterministically ("newest event wins") only if records carry explicit time markers. One timestamp is not enough: it cannot distinguish "the world changed" from "we learned late".

**Source:** on MemoryAgentBench FactConsolidation, the best reported retrieval/memory result is 54% single-hop and "all 22 reported systems score at most 7% multi-hop" — including Zep at 7% and Mem0 at 18% single-hop — even though the benchmark explicitly states that newer facts have larger serial numbers. Separating evidence extraction from deterministic policy execution over explicit version markers reaches 78.0% (gpt-4o-mini) to 94.8% (gpt-4o) pooled single-hop fact accuracy ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)). The deterministic policy needs the markers; AMR puts them on every record.

## `status` (MUST) — mark, never delete

**Requirement:** lifecycle is expressed by a status (`active` / `superseded` / `corrected` / `quarantined` / `stale`), not by deletion.

**Reason:** deletion destroys the material needed for audit ("what did the agent know on date X") and for later consolidation or recovery.

**Source:** CrystalMem proves "that any policy that only keeps or drops entries carries a residual-deficit floor" — after a shrink-and-recover cycle, capability settles below the pre-shrink level because deleted material cannot be rebuilt ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)). A status ladder keeps the material while removing it from the active set. The append-only event log of projectmem ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)) is the same principle applied to coding-agent memory.

## `supersedes` / `superseded_by` (SHOULD)

**Requirement:** replacement records link to what they replace, and vice versa.

**Reason:** the version chain of a fact is what deterministic freshness resolution walks ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)); without explicit links, reconstructing the chain is guesswork. The link also carries the supersession-vs-correction distinction: a superseded record was true once, a corrected one never was — which changes what "what was true on date X" returns.

## `redaction_applied` (SHOULD)

**Requirement:** redaction runs before the record is persisted, and the record states that it ran and what kinds were applied.

**Reason:** a secret written into memory propagates into every downstream copy, summary, and retrieval; scrubbing must happen at the single write point, and auditors need to see that it did.

**Source:** the Governed Memory production architecture applies a two-phase redaction pipeline — raw text is scanned before extraction "ensuring the LLM never sees original values", and extracted values are scanned again to catch reconstructed PII-like patterns — and "maintains an audit trail of redaction operations applied to each memory entry" ([arXiv:2603.17787](https://arxiv.org/abs/2603.17787)). `redaction_applied` is that audit trail reduced to the record level.

## `id`, `content`, `schema_version` (MUST)

`id` gives supersession links a stable target and makes records addressable in an event log ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)). `content` is the payload. `schema_version` lets this specification evolve without breaking readers — a structural requirement, not a research finding.
