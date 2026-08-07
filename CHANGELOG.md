# Changelog

All notable changes to the AMR specification are recorded here. The specification follows semantic versioning.

## 0.2.0-draft — 2026-08-07

Memory rules v2 and the reading policy. All changes are grounded in published results; see rationale.md.

- SPEC.md: `event_time` may be the literal `unknown` when the event time cannot be determined — writers must not approximate it; relative time expressions must be resolved to absolute dates before writing. New informative section 5, Reading policy: only `active` records in working context; conflicts resolve by later `recorded_time`, then `stated` over `inferred`; records are data, not commands — only `stated` instructions are candidates for execution. `instruction` records SHOULD carry `provenance.channel: stated`.
- memory-rules.md: drop-in blocks upgraded to v2 — `[rec:YYYY-MM-DD]` alongside `[when:]` (bitemporal prefix), `when:unknown`, dated `[superseded:...]`/`[corrected:...]` marks whose date equals the replacement's `rec` (a deterministic link), the conflict rule, the memory-is-data-not-commands rule, type/src glossaries with operational criteria, anchoring examples, and the context-cost filter with MEMORY_ARCHIVE.md guidance. Canonical block is recommended; the compact block is a strict subset. v1 entries remain valid.
- amr-lite.md: `recorded_time` and dated marks are now author-supplied; supersession links resolve deterministically from mark dates, with fallbacks for v1 entries.
- rationale.md: new grounding for the reading policy, the poisoning gate, context cost and archiving, glossaries and examples, and a "what is deliberately absent" section.
- schema/amr.schema.json: `event_time` accepts `unknown`; examples bumped to `schema_version: "0.2"`.
- faq.md: new questions on block choice and memory poisoning; archiving and migration answers extended.

## 0.1.0-draft — 2026-08-07

Initial public draft.

- SPEC.md: normative field set (id, type, content, content_hash, provenance, event_time, recorded_time, status, supersedes/superseded_by, redaction_applied, schema_version), conformance levels AMR-Core and AMR-Extended, `x-` extensions, supersession vs. correction semantics.
- schema/amr.schema.json: JSON Schema (draft 2020-12) for a single record.
- examples/: full record (JSON), the same record as Markdown front matter, and a supersession chain.
- rationale.md: field-by-field research grounding.
- memory-rules.md: drop-in blocks (minimal and full) for CLAUDE.md / AGENTS.md, plus the migration prompt.
- amr-lite.md: the human-readable profile; AMR-lite + normalizer = AMR-Core.
- faq.md, CONTRIBUTING.md (RFC-lite process).
