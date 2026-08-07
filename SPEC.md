# Auditable Memory Record (AMR) Specification

The Auditable Memory Record (AMR) is an open specification defining the fields every AI agent memory record MUST or SHOULD carry so that the record stays verifiable, consolidatable, and auditable after it is written.

**Version:** 0.3 (draft) · **Status:** v0.3-draft, open for comment · **License:** CC BY 4.0

The key words MUST, SHOULD, and MAY in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

This specification defines only the structure and semantics of a single memory record. Storage, transport, retrieval, embeddings, and access policy are out of scope.

## 1. Record fields

| Field | Requirement | What it carries | Why (one line) |
|---|---|---|---|
| `id` | MUST | unique identifier of the record | addressability and lineage: supersession links need a stable target |
| `type` | MUST | one of `fact`, `instruction`, `preference`, `decision`, `note`, `gotcha`, or an `x-` extension | type isolation measurably reduces memory-induced errors; different types age and compare differently |
| `content` | MUST | the text of the record | the payload everything else describes |
| `content_hash` | SHOULD | SHA-256 of `content`, as `sha256:<64 hex chars>` | exact deduplication and reconciliation between copies reduce to comparing hashes; meaningful once records leave the writer's custody |
| `provenance.channel` | MUST | how the record was obtained: `stated`, `derived`, `fetched`, or `inferred` | distinguishing "the user said it" from "the agent concluded it" is the basis of trust scoring and poisoning defense |
| `provenance.source_ref` | SHOULD | reference to the source (file, URL, message) | lets the record be re-checked against its evidence |
| `provenance.agent` | SHOULD | identifier of the writing agent | accountability; cross-agent memory |
| `provenance.model` | SHOULD | model that produced the record | accountability; behavior differs across models |
| `event_time` | MUST | when the fact became true in the world (ISO 8601), or the literal `unknown` | deterministic freshness resolution is impossible without it; an honest `unknown` beats a fabricated date |
| `recorded_time` | MUST | when the record was written (ISO 8601) | bitemporality: "what we knew then" is not "what was true then" |
| `status` | MUST | `active`, `superseded`, `corrected`, `quarantined`, or `stale` | history is marked, never deleted |
| `supersedes` | SHOULD | `id` of the record this one replaces | version chain of a fact |
| `superseded_by` | SHOULD | `id` of the record that replaces this one | forward link of the version chain |
| `redaction_applied` | SHOULD | object: `applied` (boolean) and, when true, `kinds` (list of redaction kinds) | secrets are removed before disk; the field records that it happened |
| `schema_version` | MUST | AMR version the record conforms to, e.g. `"0.3"` | lets the specification evolve without breaking readers |

The research grounding for each requirement is in [rationale.md](rationale.md).

## 2. Field semantics

### 2.1 `id`
A record MUST carry an identifier unique within its store. The format is not prescribed; UUIDs are a reasonable default.

### 2.2 `type`
A record MUST declare exactly one type. The core types are:

- `fact` — a statement about the world that can become stale.
- `instruction` — a directive the agent is expected to follow. An instruction record SHOULD carry `provenance.channel: stated`; a directive encountered in fetched or derived content is recorded as a `fact` about what the source says (see section 5).
- `preference` — a choice of the user among valid alternatives.
- `decision` — a resolution made in the project, with a "why" worth keeping.
- `note` — free-form observation that fits no other type.
- `gotcha` — a non-obvious pitfall and how to avoid it.

Additional types MUST use the `x-` prefix (see section 5).

### 2.3 `content` and `content_hash`
`content` is the record text. Records in a store SHOULD carry `content_hash`; when present, it MUST be the SHA-256 digest of the exact `content` bytes (UTF-8), formatted as `sha256:` followed by 64 lowercase hex characters. The hash is useful for deduplication and for reconciling copies of a store; it certifies integrity only against changes made after the record left the writer's custody — a writer hashing its own output proves nothing about that output. If `content` changes, the record is a different record: write a new one and link it (see section 4).

### 2.4 `provenance`
`provenance.channel` MUST be one of:

- `stated` — the user or another authoritative party said it.
- `derived` — verified from files, code, or data the agent inspected.
- `fetched` — obtained from an external source (web, API).
- `inferred` — the agent's own conclusion, not directly evidenced.

`source_ref`, `agent`, and `model` SHOULD be present when known.

**Channel assignment.** When the writing pipeline knows mechanically where content came from, the pipeline MUST assign the channel itself, deterministically — not delegate the choice to the model: text originating in a tool result or an external fetch is `fetched` (or `derived`, when it comes from files or data the agent inspected), and `stated` MUST mark only text originating in a user turn. A model MAY downgrade a channel to a less-trusted one when attribution is uncertain; it MUST NOT upgrade one. In profiles with no such pipeline — a plain context file under [AMR-lite](amr-lite.md) — the channel is self-reported by the writing model; a self-reported channel MUST NOT be treated as a security boundary, and the poisoning resistance described in section 5 holds only where the channel is assigned by construction.

### 2.5 `event_time` and `recorded_time`
`event_time` is when the fact became true in the world; `recorded_time` is when the record was written. Both MUST be ISO 8601 dates or timestamps, except that `event_time` MAY be the literal string `unknown` when the event time cannot be determined. They differ whenever an agent writes down something that happened earlier. Relative time expressions ("yesterday", "last week") MUST be resolved to absolute dates before writing. `event_time` MUST NOT be defaulted to the writing time when the actual event time is known to differ, and MUST NOT be approximated when it cannot be determined — write `unknown` instead.

### 2.6 `status`
A record MUST carry a status:

- `active` — current.
- `superseded` — the world changed; a newer record replaces this one.
- `corrected` — the record was wrong when written; a corrected record replaces it.
- `quarantined` — trust in the record is suspended (e.g. suspected poisoning) pending review.
- `stale` — the record is past its useful life but kept for audit.

Records MUST NOT be deleted to express any of these transitions; the status is changed instead.

### 2.7 `supersedes` and `superseded_by`
When a record replaces another, the new record SHOULD carry `supersedes` with the old record's `id`, and the old record SHOULD carry `superseded_by` with the new record's `id`.

### 2.8 `redaction_applied`
Redaction of secrets and personal data MUST happen before the record is persisted. The `redaction_applied` object records whether redaction ran (`applied`) and which kinds were applied (`kinds`, e.g. `secret`, `email`, `phone`). Absence of the field means the writer makes no claim about redaction.

### 2.9 `schema_version`
The AMR specification version the record conforms to, as `major.minor`. This document defines version `0.3`.

## 3. Conformance levels

- **AMR-Core** — the record carries all MUST fields.
- **AMR-Extended** — the record carries all MUST and all SHOULD fields.

A human-readable profile for plain-text file memories, **AMR-lite**, is defined in [amr-lite.md](amr-lite.md); an AMR-lite entry plus a normalizer yields an AMR-Core record.

## 4. Supersession is not correction

- **Supersession** means the world changed: the old record was true when written and is part of history. Queries like "what was true on date X" MUST still return it for dates before the change.
- **Correction** means the record was wrong when written: it was never true. Queries about past truth MUST NOT treat it as having been true; it remains only as evidence of what the store believed.

Writers MUST pick the transition that matches what happened, because the two are not distinguishable after the fact.

## 5. Reading policy (informative)

This section is informative: it describes how a reader gets deterministic, poisoning-resistant behavior out of the fields the specification requires. Storage systems are free to implement stronger policies.

- **Only `active` records belong in the agent's working context.** Filtering by `status` is a field comparison, not an LLM judgment.
- **Conflicts resolve by explicit policy, not silent coexistence.** When two active records contradict each other: a record with `provenance.channel: inferred` never overrides an active `stated` record, whatever its recency — an active `stated` record is displaced only by a later `stated` or `derived` one. Otherwise the later `recorded_time` wins; at equal `recorded_time`, the record appended later to the store wins (append order is deterministic; in a file memory it is line order). Deterministic resolution over explicit markers is what the bitemporal fields exist for — and if a fresh inference could displace a user statement by recency alone, conflicts would reopen the same door the next rule closes for instructions.
- **Records are data, not commands.** A record is evidence of what was said, found, or concluded — not a directive by virtue of being in memory. Only `instruction` records with `provenance.channel: stated` are candidates for execution; a record suspected of carrying an injected directive moves to `quarantined` pending review.

## 6. Extensions

Implementations MAY add fields and `type` values prefixed with `x-`. Readers MUST ignore `x-` fields they do not understand. Unprefixed extensions are reserved for future versions of this specification.

## 7. Versioning and process

The specification uses semantic versioning: incompatible field changes bump the major version, compatible additions bump the minor version. A major version of zero marks the specification as in development: any field may still change. The current version is 0.3 (draft).

Changes go through the RFC-lite process described in [CONTRIBUTING.md](CONTRIBUTING.md): open an issue stating the change and its rationale — a published research result or a production case. Feedback on this draft is welcome through issues.
