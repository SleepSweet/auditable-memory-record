# Changelog

All notable changes to the AMR specification are recorded here. The specification follows semantic versioning.

## 0.1.0-draft — 2026-08-07

Initial public draft.

- SPEC.md: normative field set (id, type, content, content_hash, provenance, event_time, recorded_time, status, supersedes/superseded_by, redaction_applied, schema_version), conformance levels AMR-Core and AMR-Extended, `x-` extensions, supersession vs. correction semantics.
- schema/amr.schema.json: JSON Schema (draft 2020-12) for a single record.
- examples/: full record (JSON), the same record as Markdown front matter, and a supersession chain.
- rationale.md: field-by-field research grounding.
- memory-rules.md: drop-in blocks (minimal and full) for CLAUDE.md / AGENTS.md, plus the migration prompt.
- amr-lite.md: the human-readable profile; AMR-lite + normalizer = AMR-Core.
- faq.md, CONTRIBUTING.md (RFC-lite process).
