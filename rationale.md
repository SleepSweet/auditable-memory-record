# Field-by-field rationale

This document states, per field of the [AMR specification](SPEC.md) and per rule of the [drop-in block](memory-rules.md): the requirement, the reason, and the evidence — with its strength graded honestly. Each section carries an evidence label:

- **direct** — the source measures the mechanism the requirement relies on;
- **transferred** — the source establishes the principle in a different setting; carrying it here is our judgment, and the label says so;
- **engineering decision** — no research result behind it; the requirement stands on stated engineering reasoning alone.

All numbers below are quoted as published; each citation was checked against the paper before inclusion. Not every requirement traces to a published result — the labels say which do, and where a published source also bounds a claim, the bound is quoted next to the claim.

## Why a write-time standard at all

Context files are not automatically useful. An evaluation of repository-level context files (AGENTS.md and similar) found that providing them "does not generally improve task success rates, while increasing inference cost by over 20% on average" — while also finding that "instructions in the context files are well followed by coding agents" ([arXiv:2602.11988](https://arxiv.org/abs/2602.11988)). The lesson for memory files is direct: agents will follow what you write, and unhelpful content costs real money — so what goes into a memory entry at write time decides whether the file pays for itself.

Write time is also where the damage happens. In MemGuard's analysis, "unverifiability errors are predominantly associated with write-time contamination (97.7%)" ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) — records that cannot be checked later are almost always born broken, not broken later. A standard that only governs retrieval or consolidation arrives too late; AMR therefore standardizes the record at the moment it is written.

The survey literature on agent accountability converges on the same record-level minimum. A survey of evidence tracing and execution provenance in LLM agents taxonomizes what an auditable agent system must capture per unit of evidence — the source it came from, when it was captured, and how it was subsequently used and updated ([arXiv:2606.04990](https://arxiv.org/abs/2606.04990)). AMR's `provenance`, the two timestamps, and the status-plus-supersession lifecycle are that minimum expressed as record fields.

**Evidence: direct** for the two measured results (context-file cost and write-time contamination are measured on the artifact class itself); the survey mapping is transferred.

## `type` (MUST)

**Requirement:** every record declares exactly one type.

**Reason:** different types age differently (a `fact` can go stale; a `preference` rarely does), compare differently, and must not be used as interchangeable evidence.

**Source:** MemGuard identifies "heterogeneous memory contamination" — functionally distinct memories collapsed into a shared space and misused as interchangeable evidence — and fixes it by assigning "each memory an explicit functional role at write time". The type-aware framework "improves memory reliability by up to 28.27% while retrieving up to 5.8x fewer memory tokens than prior methods" ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)).

**Evidence: direct for the mechanism, transferred to this profile.** The number was measured inside MemGuard's own memory framework, not for a prefix tag in a context file; what AMR carries over is the measured principle — explicit functional role assigned at write time — not the measured percentage. This is also why `type` is MUST while `content_hash` is not: it is the best-evidenced field in the record, and it costs the writer nothing — in every profile the author supplies it directly (in AMR-lite it is simply part of the prefix), whereas the hash can only be filled in downstream by a normalizer certifying its own reading.

## `content_hash` (SHOULD)

**Requirement:** SHA-256 of the record content, stored with the record — an AMR-Extended field for store-grade records.

**Reason:** exact deduplication, incremental re-processing, and reconciliation between copies of a store all reduce to comparing hashes, without an LLM call.

**Evidence: engineering decision.** No research result backs this field; hashing is standard engineering for content addressing, stated as such. Its honest limits are also why it is not in the MUST set: a hash certifies integrity only against changes made after the record leaves the writer's custody. A normalizer hashing lines it has just read from a text file confirms the integrity of its own reading — nothing about tampering — so requiring the hash of every record would calibrate the core to databases while the flagship profile is a text file. Stores exchanging or replicating records SHOULD carry it; AMR-Core does not demand it.

## `provenance.channel` (MUST), `source_ref` / `agent` / `model` (SHOULD)

**Requirement:** every record states how it was obtained (`stated` / `derived` / `fetched` / `inferred`); when known, also from what source and by which agent and model.

**Reason:** "the user said it" and "the agent concluded it" deserve different trust, different staleness policies, and different treatment under suspected poisoning. Without the channel recorded at write time, this distinction is unrecoverable later.

**Source:** 97.7% of unverifiability errors are associated with write-time contamination ([arXiv:2605.28009](https://arxiv.org/abs/2605.28009)) — verifiability must be captured when the record is written. An append-only, provenance-carrying event log as the basis for auditable agent memory is demonstrated by projectmem, whose "immutable log also serves as a provenance trail for reproducible, auditable AI-assisted development" ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)).

**Evidence: direct** for capturing provenance at write time; **engineering decision** for the specific four-value channel taxonomy — no study compares channel vocabularies. A related requirement follows from the threat model rather than a paper: a provenance tag chosen by the model it is meant to constrain is a self-report, so where the pipeline mechanically knows the channel it must assign it itself ([SPEC.md, section 2.4](SPEC.md#24-provenance)); see the poisoning section below for what this does and does not buy.

## `event_time` + `recorded_time` (MUST)

**Requirement:** two timestamps per record — when the fact became true in the world, and when it was written down.

**Reason:** conflict resolution between contradictory facts can be done deterministically ("newest event wins") only if records carry explicit time markers. One timestamp is not enough: it cannot distinguish "the world changed" from "we learned late".

**Source:** on MemoryAgentBench FactConsolidation, the best reported retrieval/memory result is 54% single-hop and "all 22 reported systems score at most 7% multi-hop" — including Zep at 7% and Mem0 at 18% single-hop — even though the benchmark explicitly states that newer facts have larger serial numbers. Separating evidence extraction from deterministic policy execution over explicit version markers reaches 78.0% (gpt-4o-mini) to 94.8% (gpt-4o) pooled single-hop fact accuracy ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)). The deterministic policy needs the markers; AMR puts them on every record.

The two-axis model is standard practice, not an AMR invention. Zep's temporal knowledge graph — the most-cited temporal agent memory architecture — tracks both valid time (when the fact held in the world) and ingestion time per fact, and resolves contradictions by invalidating, never deleting, the outdated edge ([arXiv:2501.13956](https://arxiv.org/abs/2501.13956)); the same bitemporal model was standardized for databases in SQL:2011. Two consequences follow for writers. First, relative time expressions ("yesterday", "two weeks ago") must be resolved to absolute dates at write time — the reference point is lost by the next session, which is why temporal extraction to absolute dates is an explicit pipeline step in these systems. Second, `event_time` may honestly be `unknown`: valid time is sometimes undeterminable, while recording time never is, and a fabricated approximation silently corrupts the deterministic resolution that the field exists to enable.

**Honest bound.** The same paper reports the other side of its result, and AMR quotes it rather than hides it. On LongMemEval — general conversational QA — structured deterministic assembly shows no significant gain over direct judgment (26/45 vs 29/45 correct, paired exact McNemar p=0.45), and the authors state that "the evidence therefore supports a narrower claim": the assembly policy is useful for current-value questions, while broader conversational QA requires question-type-aware composition. Multi-hop consolidation reaches 30.2–51.5% pooled — far above the ≤7% ceiling of the 22 prior systems, far below solved. The claim AMR inherits is accordingly narrow: explicit per-record time markers make current-value conflict resolution deterministic. Nothing in this specification promises general QA gains.

**Evidence: direct** — the consolidation result is measured on explicit version markers, the mechanism these fields provide; the quoted bound is part of the same measurement.

## `status` (MUST) — mark, never delete

**Requirement:** lifecycle is expressed by a status (`active` / `superseded` / `corrected` / `quarantined` / `stale`), not by deletion.

**Reason:** deletion destroys the material needed for audit ("what did the agent know on date X") and for later consolidation or recovery.

**Source:** CrystalMem proves "that any policy that only keeps or drops entries carries a residual-deficit floor" — after a shrink-and-recover cycle, capability settles below the pre-shrink level because deleted material cannot be rebuilt ([arXiv:2608.00303](https://arxiv.org/abs/2608.00303)). A status ladder keeps the material while removing it from the active set. The append-only event log of projectmem ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)) is the same principle applied to coding-agent memory.

**Evidence: transferred.** CrystalMem's theorem is about keep-or-drop policies over model memory in general; forbidding line deletion in a memory file carries the principle to this artifact class — the floor itself was not measured on context files.

## `supersedes` / `superseded_by` (SHOULD)

**Requirement:** replacement records link to what they replace, and vice versa.

**Reason:** the version chain of a fact is what deterministic freshness resolution walks ([arXiv:2606.01435](https://arxiv.org/abs/2606.01435)); without explicit links, reconstructing the chain is guesswork. The link also carries the supersession-vs-correction distinction: a superseded record was true once, a corrected one never was — which changes what "what was true on date X" returns.

**Source:** work on governed shared memory for multi-agent LLM systems names temporal supersession and provenance tracking among the primitives production memory requires, precisely because without explicit supersession the alternative is contradictory records silently coexisting ([arXiv:2606.24535](https://arxiv.org/abs/2606.24535)). In the [AMR-lite](amr-lite.md) profile the same link is carried without record IDs: the date inside a `[superseded:...]` mark equals the replacement entry's `rec`, making the replacement findable by a deterministic scan — a deliberate 80%-of-the-benefit trade against introducing per-line identifiers into a plain-text file.

**Evidence: transferred** for the primitive (named as required in a different, multi-agent setting); **engineering decision** for the dated-mark link mechanism in the lite profile.

## `redaction_applied` (SHOULD)

**Requirement:** redaction runs before the record is persisted, and the record states that it ran and what kinds were applied.

**Reason:** a secret written into memory propagates into every downstream copy, summary, and retrieval; scrubbing must happen at the single write point, and auditors need to see that it did.

**Source:** the Governed Memory production architecture applies a two-phase redaction pipeline — raw text is scanned before extraction "ensuring the LLM never sees original values", and extracted values are scanned again to catch reconstructed PII-like patterns — and "maintains an audit trail of redaction operations applied to each memory entry" ([arXiv:2603.17787](https://arxiv.org/abs/2603.17787)). `redaction_applied` is that audit trail reduced to the record level.

**Evidence: transferred** — a production architecture's pipeline-level practice, reduced to a record-level field by our judgment.

## `id`, `content`, `schema_version` (MUST)

`id` gives supersession links a stable target and makes records addressable in an event log ([arXiv:2606.12329](https://arxiv.org/abs/2606.12329)). `content` is the payload. `schema_version` lets this specification evolve without breaking readers.

**Evidence: engineering decision** — structural requirements, not research findings.

## Reading policy: conflict resolution ([SPEC section 5](SPEC.md#5-reading-policy-informative))

**Requirement:** when two active records contradict each other: `inferred` never overrides an active `stated` record, whatever its recency — an active `stated` record is displaced only by a later `stated` or `derived` one. Otherwise the later `recorded_time` wins; at equal `recorded_time`, the record appended later wins (append order; in a file memory, line order).

**Reason:** an append-only store guarantees that conflicts will eventually exist — a writer will sooner or later forget to mark the old record. Without an explicit read-time rule, what the agent answers in that situation is nondeterministic. Two choices inside the rule need their own why. Provenance outranks recency for the stated/inferred pair because the trust model must be self-consistent: if recency alone could displace a user statement with a fresh inference, the trust ordering the poisoning gate depends on would break — the agent's own conclusion replacing what the user actually said is precisely a failure class this format exists to remove. The same boundary holds for `fetched`: an external source, however fresh, does not outrank what the user said — which is why the rule names `stated` and `derived` as the only channels that displace a stated record, not merely "anything newer than inferred". The tie-break rests on append order because it is the one deterministic order an append-only store already carries; demanding finer timestamps would invite fabricated precision (see the granularity section below).

**Evidence: engineering decision, informed by [arXiv:2606.24535](https://arxiv.org/abs/2606.24535)** — the governed-shared-memory work resolves contradictory writes through temporal ordering plus policy-aware resolution rules instead of silent coexistence; the specific precedence AMR picks (provenance before recency for the stated/inferred pair, append order as tie-break) is our design, not a measured result.

## Reading policy: memory is data, not commands

**Requirement:** a record is evidence, not a directive by virtue of being in memory. Only `instruction` records with `provenance.channel: stated` are candidates for execution; a directive found in fetched or derived content is recorded as a `fact` about what the source says.

**Reason:** memory poisoning turns a one-shot prompt injection into a persistent compromise — a malicious "instruction", once written (for example, from a web page through session summarization), executes in every future session.

**Source:** a systematic study of memory poisoning (MPBench) identifies four memory write channels and six attack classes, and finds that "existing prompt injection defenses fail to cover memory poisoning" ([arXiv:2606.04329](https://arxiv.org/abs/2606.04329)). Poisoned memories succeed in 60–89% of cases across tested models in sleeper-poisoning evaluations ([arXiv:2605.15338](https://arxiv.org/abs/2605.15338)). The attack is not theoretical: Unit 42 demonstrated a working proof of concept against Amazon Bedrock Agents — a malicious webpage poisons the agent's long-term memory via session summarization, and later sessions silently exfiltrate conversation history ([Unit 42](https://unit42.paloaltonetworks.com/indirect-prompt-injection-poisons-ai-longterm-memory/)).

The rule is cheap because the marking already exists: `provenance.channel` is captured at write time, so refusing executability to non-`stated` channels is one line of policy over a field the record carries anyway. The constructive form — record the directive as a `fact` about what the source says — loses no information; only executability is lost. Honest calibration, in two steps. First, this is a prompt-level mitigation that closes the cheapest attack channel, not a complete defense — no prompt-level measure is; retrieval-time filters and isolation remain the memory system's responsibility ([Mem0 security guide](https://mem0.ai/blog/ai-memory-security-best-practices)). Second, and sharper: the gate is only as trustworthy as the channel tag it reads, and in AMR-lite that tag is self-reported by the very model an attacker targets. A compromised writer can label injected content `stated`. The gate therefore becomes a guarantee only where the channel is assigned by the pipeline, deterministically — tool-result text marked `fetched` by code, `stated` reserved for user-turn text — which the specification makes a requirement wherever such a pipeline exists ([SPEC.md, section 2.4](SPEC.md#24-provenance)). AMR-lite in a plain context file has no such pipeline and provides hygiene, not poisoning guarantees; the specification says so in as many words.

**Evidence: direct** for the attack and its coverage gap (measured and demonstrated); **engineering decision** for the gate itself — its effectiveness as a defense has not been independently measured.

## Context cost: the value filter and archiving

**Requirement (rules block):** write only entries worth their context cost in every future session; periodically move `[superseded:...]`/`[corrected:...]` lines to an archive file that is not auto-loaded. Deletion stays forbidden — archiving moves records between files, it does not destroy them.

**Reason:** a file memory is read at the start of every session and paid for in context tokens every time. Dead lines accumulate under an append-only discipline and tax every future session.

**Source:** Anthropic's guidance is direct — memory files load every session, and "shorter files produce better adherence" ([Claude Code memory documentation](https://code.claude.com/docs/en/memory)). The degradation is measured independently: attention to the middle of long contexts collapses ("Lost in the Middle", [arXiv:2307.03172](https://arxiv.org/abs/2307.03172)), and across 18 models — including Claude 4, GPT-4.1, and Gemini 2.5 — performance "grows increasingly unreliable as input length grows" even on simple tasks ([Context Rot, Chroma, 2025](https://research.trychroma.com/context-rot)). Archiving reconciles this with the no-deletion requirement (the residual-deficit floor of [arXiv:2608.00303](https://arxiv.org/abs/2608.00303)): history stops costing tokens without being destroyed.

**Evidence: direct** for the vendor guidance (it is about these files); **transferred** for the long-context measurements, which are about long inputs in general.

## Glossaries and examples in the block

**Requirement (rules block):** the canonical block defines each `type` and `src` value with an operational criterion (for `derived`: "I can point to the file/line proving it"), and ends with a minimal set of examples, including a supersede-and-replacement pair.

**Reason:** a taxonomy without definitions degrades into arbitrary tag choice — and two of the block's boundaries carry weight: `instruction` vs `preference` sits under the poisoning gate, and `derived` vs `inferred` sets priority in conflict resolution.

**Source:** a controlled study of style control in multi-turn code generation finds that abstract directives produce large initial effects and retain their discipline over subsequent turns, examples alone produce modest initial effects with no such discipline, and the combination is strongest ([arXiv:2511.13972](https://arxiv.org/abs/2511.13972)). Hence the division of labor: the rules carry the semantics, and a few examples anchor the surface format — the pair of linked lines demonstrates the one lifecycle mechanic (mark date = replacement's `rec`) that a single line cannot show. Anthropic's documentation makes the same point for context files: the more specific and concise the instruction, the more consistently it is followed ([memory documentation](https://code.claude.com/docs/en/memory)).

**Evidence: transferred** — the controlled study is about style control in code generation; carrying its directives-plus-examples finding to a memory-rules block is our judgment.

## Dates, not timestamps, in AMR-lite

**Requirement (lite profile):** `when`, `rec`, and mark dates are calendar dates (`YYYY-MM-DD`), not timestamps; within a day, order is carried by file position.

**Reason:** a date is the maximum granularity the lite writer can supply honestly. A model writing into a context file reliably knows the current date but not a trustworthy clock or timezone; a fabricated `T14:37:00+03:00` is exactly the kind of silently corrupting precision the `when:unknown` rule exists to forbid, and a wrong order is worse than a coarse one — the coarse date honestly says "same day", the wrong timestamp silently picks the wrong record. Dates also keep the per-entry prefix cost low (paid in every future session — see the context-cost section above), and the date inside a supersession mark doubles as a robust scan link, where second-level precision would make the link brittle. Within-day ordering needs no clock at all: entries are appended, so file position is a deterministic tie-break the reading policy already uses. Full AMR records written by a harness with a real clock carry ISO 8601 timestamps ([SPEC.md, section 2.5](SPEC.md#25-event_time-and-recorded_time)).

**Evidence: engineering decision.**

## What is deliberately absent

Every line of a rules block competes for the agent's limited instruction-following budget, so everything that does not pay for itself in a plain-text profile was rejected:

- **Per-line record IDs** — the date inside a supersession mark is already a deterministic, scan-reachable reference to the replacement; a new identity per line adds ceremony without new capability. Full AMR records (JSON stores) do carry `id`.
- **A confidence field** — present in academic provenance architectures ([arXiv:2606.04990](https://arxiv.org/abs/2606.04990)), but an LLM fills it arbitrarily at write time; a field whose value cannot be validated is noise wearing the costume of rigor.
- **LLM-assisted deduplication and retrieval-time poisoning filters** — properties of an external memory system, not implementable by a context-file block. Their absence marks the boundary of the format, not a gap in it.
- **Conformance tooling** — a validator or normalizer is an implementation, and this repository is the standard: specification text and examples only, no code. The JSON Schema is the machine-checkable surface the specification itself provides; executable checking belongs to the implementations listed in the [README](README.md#implementations).
