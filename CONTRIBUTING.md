# Contributing

The specification changes through an RFC-lite process. The bar is deliberately simple and deliberately evidence-based.

## Proposing a change

1. Open an issue describing the change: which field or rule, what changes, and — required — a **Rationale** section containing one of:
   - a published research result (link + the specific finding), or
   - a production case (what happened, at what scale, and why the current specification made it worse).

   Proposals without a rationale of one of these two kinds are closed as-is; this mirrors the rule the specification itself follows — every field in [SPEC.md](SPEC.md) is grounded in [rationale.md](rationale.md).

2. Discussion happens on the issue. When there is rough consensus, open a PR referencing the issue.

3. The PR updates, together: SPEC.md, schema/amr.schema.json, the examples, and rationale.md — the four must stay field-for-field identical — plus a CHANGELOG.md entry.

## Versioning

Semantic versioning of the specification: incompatible field changes bump the major version, compatible additions bump the minor version. Experimental fields live under the `x-` prefix and need no process at all.

## Scope guard

The specification covers the structure and semantics of a single memory record. Storage, transport, retrieval, embeddings, and access policy are out of scope; proposals in those areas belong in implementations, not here.

## Language

All repository content, issues, and commit messages are in English.
