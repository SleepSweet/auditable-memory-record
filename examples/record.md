# The same record as Markdown front matter

The record from [record.json](record.json), represented as a Markdown file with YAML front matter — the shape file-based memories (one file per record) would use. The body is the `content`; everything else lives in the front matter.

```markdown
---
id: ff85e2e5-407b-4cd3-8f49-b8c7d2ff2491
type: fact
content_hash: sha256:76735ee33de12734553bf1d4f22ab8968ace4810ffc43c4bb7f300b961e4eb63
provenance:
  channel: stated
  source_ref: chat message from the project owner, 2026-07-14
  agent: cli-coding-agent
  model: example-model-1
event_time: 2026-07-14
recorded_time: 2026-07-14T18:22:05Z
status: active
redaction_applied:
  applied: false
schema_version: "0.3"
---
The staging database was migrated to PostgreSQL 17; the old MySQL instance is read-only.
```

For single-line entries inside a shared file (CLAUDE.md, AGENTS.md), use the [AMR-lite](../amr-lite.md) profile instead:

```
[type:fact] [src:stated] [when:2026-07-14] [rec:2026-07-14] The staging database was migrated to PostgreSQL 17; the old MySQL instance is read-only.
```
