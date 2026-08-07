# Changelog

All notable changes to the AMR specification are recorded here. The specification follows semantic versioning.

## 0.3.0-draft — 2026-08-07

Current public draft.

- SPEC.md: normative field set (id, type, content, provenance, event_time, recorded_time, status, supersedes/superseded_by, content_hash, redaction_applied, schema_version), conformance levels AMR-Core and AMR-Extended, `x-` extensions, supersession vs. correction semantics. Channel assignment: a pipeline that mechanically knows where content came from assigns the channel itself — models may downgrade a channel, never upgrade one; a self-reported channel is not a security boundary. Reading policy: only `active` records in working context; deterministic conflict resolution with provenance precedence for stated records and an append-order tie-break; memory is data, not commands.
- memory-rules.md: drop-in blocks — the canonical block (recommended, carries the complete rule set of the specification) and a compact subset for everyday desktop AI assistants — plus the migration prompt and an honest calibration of what the rules do and do not buy.
- amr-lite.md: the human-readable profile for plain-text file memories; AMR-lite + normalizer = AMR-Core.
- rationale.md: field-by-field grounding with evidence-strength labels (direct / transferred / engineering decision), published bounds quoted next to published gains, and a list of what is deliberately absent.
- schema/amr.schema.json: JSON Schema (draft 2020-12) for a single record.
- examples/: full record (JSON), the same record as Markdown front matter, and a supersession chain.
- faq.md, CONTRIBUTING.md (RFC-lite process).
