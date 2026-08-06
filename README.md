# Auditable Memory Record (AMR) — an open specification for auditable AI agent memory records

The Auditable Memory Record (AMR) is an open specification defining the fields every AI agent memory record should carry — type, content hash, provenance channel, event time and record time, redaction status — together with the published research rationale for each field.

AI agent memory degrades over time: duplicates, contradictions, stale facts, and poisoned entries accumulate as agents write to AGENTS.md, CLAUDE.md, auto-memory files, and memory stores, and this measurably hurts task success and token cost. Most of that damage is either caused or made unfixable at write time — records are stored without type, provenance, or timestamps, so no later tool can tell fresh from stale or trusted from injected. AMR standardizes what to write so that memory consolidation, memory audit, and memory governance become possible after the fact. A memory system that stores these fields is AMR-compliant.

## Record fields

| Field | What it carries | Why (research rationale) |
|---|---|---|
| `type` | fact, instruction, preference, decision, definition, procedure, or observation | Typed records enable type-scoped deduplication and per-type processing; type isolation measurably reduces memory-induced hallucination (arXiv:2605.28009). |
| `content_hash` | hash of the normalized record content | Exact deduplication, incremental re-processing, and tamper evidence all reduce to comparing hashes. |
| `provenance.channel` | how the record was obtained: stated, verified, fetched, or inferred | The large majority of confabulated memories are born at write time; recording the channel at write time makes trust computable later (arXiv:2605.28009). |
| `event_time` + `record_time` | when the fact was true in the world vs. when it was written down (bitemporal) | Conflict resolution done deterministically over version markers beats asking an LLM to track freshness; it requires both timestamps to distinguish "the world changed" from "we were wrong" (arXiv:2606.01435). |
| `redaction_status` | whether secrets/PII were redacted, and how | Redaction must be applied and recorded at the single write point, or leaked secrets propagate into every downstream copy and summary. |
| `status` + `superseded_by` | active, superseded, corrected, quarantined, or retired; link to the replacing record | History is marked, never deleted — which is what turns a memory store into a free audit trail ("what did the agent know at date X, and who changed it, when, and why"). |

## Status

Early draft. This repository reserves the specification's public home; the full field-by-field specification text follows.

## Project family

- [sleepsweet](https://github.com/sleepsweet/sleepsweet) — engine + CLI, the reference consumer of AMR records
- [auditable-memory-record](https://github.com/sleepsweet/auditable-memory-record) — this specification
- [memory-hygiene-bench](https://github.com/sleepsweet/memory-hygiene-bench) — open benchmark measuring the cost of dirty agent memory

## License

CC BY 4.0
