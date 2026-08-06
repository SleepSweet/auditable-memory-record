# AMR-lite: the human-readable profile

AMR-lite is the representation of Auditable Memory Records for plain-text file memories (CLAUDE.md, AGENTS.md, auto-memory files): the entry author writes a single line with a metadata prefix, and a normalizer derives the remaining [AMR-Core](SPEC.md#3-conformance-levels) fields.

The entry format is defined by the drop-in block in [memory-rules.md](memory-rules.md):

```
[type:{fact|instruction|preference|decision|gotcha}] [src:{stated|derived|fetched|inferred}] [when:YYYY-MM-DD] <content>
```

## Who supplies which field

| AMR-Core field | Where it comes from in AMR-lite | Supplied by |
|---|---|---|
| `type` | `[type:...]` prefix | entry author |
| `provenance.channel` | `[src:...]` prefix | entry author |
| `event_time` | `[when:YYYY-MM-DD]` prefix | entry author |
| `content` | the line text after the prefix | entry author |
| `status` | trailing `[superseded]` / `[corrected]` mark; no mark = `active` | entry author |
| `id` | generated (e.g. UUID) | normalizer |
| `content_hash` | computed over the content | normalizer |
| `recorded_time` | file modification or commit timestamp of the line | normalizer |
| `redaction_applied` | secret/PII scan at normalization time | normalizer |
| `schema_version` | stamped with the AMR version the normalizer targets | normalizer |
| `supersedes` / `superseded_by` | resolved from marks and content similarity (best effort) | normalizer |

**The rule: AMR-lite + normalizer = AMR-Core.** A memory file whose entries follow the block, processed by a normalizer that derives the lower half of the table, yields AMR-Core records — no change to how the author writes.

Notes:

- The lite prefix lists the five most commonly written types; the full specification also defines `note` and admits `x-` extensions ([SPEC.md, section 2.2](SPEC.md#22-type)).
- A normalizer is any tool that performs the derivation — the reference implementation is listed in the [README](README.md#implementations), and the derivation is simple enough to implement in a shell script.
- When resolving replacement links, note that a correction keeps the `event_time` of the record it corrects — the world did not change, the record was wrong ([SPEC.md, section 4](SPEC.md#4-supersession-is-not-correction)). A normalizer that requires the replacement to be strictly newer will miss correction pairs; compare with "newer or equal".
