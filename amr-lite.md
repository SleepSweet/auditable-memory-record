# AMR-lite: the human-readable profile

AMR-lite is the representation of Auditable Memory Records for plain-text file memories (CLAUDE.md, AGENTS.md, auto-memory files): the entry author writes a single line with a metadata prefix, and a normalizer derives the remaining [AMR-Core](SPEC.md#3-conformance-levels) fields.

The entry format is defined by the drop-in block in [memory-rules.md](memory-rules.md):

```
[type:{fact|instruction|preference|decision|gotcha}] [src:{stated|derived|fetched|inferred}] [when:YYYY-MM-DD] [rec:YYYY-MM-DD] <content>
```

## Who supplies which field

| AMR field | Where it comes from in AMR-lite | Supplied by |
|---|---|---|
| `type` | `[type:...]` prefix | entry author |
| `provenance.channel` | `[src:...]` prefix | entry author |
| `event_time` | `[when:YYYY-MM-DD]` prefix, or `[when:unknown]` | entry author |
| `recorded_time` | `[rec:YYYY-MM-DD]` prefix; for legacy entries without it, the file modification or commit timestamp of the line | entry author (normalizer fallback) |
| `content` | the line text after the prefix | entry author |
| `status` | trailing `[superseded:YYYY-MM-DD]` / `[corrected:YYYY-MM-DD]` mark; no mark = `active` | entry author |
| `id` | generated (e.g. UUID) | normalizer |
| `content_hash` (Extended) | computed over the content, for deduplication and reconciliation between copies | normalizer (optional) |
| `redaction_applied` (Extended) | secret/PII scan at normalization time | normalizer |
| `schema_version` | stamped with the AMR version the normalizer targets | normalizer |
| `supersedes` / `superseded_by` | resolved from the date in the mark, which equals the replacement entry's `rec`; content similarity for undated legacy marks (best effort) | normalizer |

**The rule: AMR-lite + normalizer = AMR-Core.** A memory file whose entries follow the block, processed by a normalizer that derives the lower half of the table, yields AMR-Core records — no change to how the author writes.

Notes:

- The lite prefix lists the five most commonly written types; the full specification also defines `note` and admits `x-` extensions ([SPEC.md, section 2.2](SPEC.md#22-type)).
- A normalizer is any tool that performs the derivation — the reference implementation is listed in the [README](README.md#implementations), and the derivation is simple enough to implement in a shell script.
- When resolving replacement links, note that a correction keeps the `event_time` of the record it corrects — the world did not change, the record was wrong ([SPEC.md, section 4](SPEC.md#4-supersession-is-not-correction)). A normalizer that requires the replacement to be strictly newer will miss correction pairs; compare with "newer or equal".
- The date in a `[superseded:...]`/`[corrected:...]` mark is a reference, not just a timestamp: it equals the `rec` of the replacement entry, so the replacement is found by a deterministic scan for that `rec` — no similarity heuristics needed. Undated marks and missing `[rec:...]` prefixes identify legacy entries written before the block was adopted; they remain valid, with the normalizer falling back to file history and content similarity.
- The `src` tag in a lite entry is self-reported by the writing model. AMR-lite therefore carries provenance for hygiene and audit, not as a security boundary: the poisoning resistance of the reading policy holds only where the channel is assigned by the pipeline ([SPEC.md, section 2.4](SPEC.md#24-provenance)) — which a plain context file cannot do. A normalizer that validates or assigns the channel from session evidence is what upgrades the tag from self-report to guarantee.
- Prefix dates are calendar dates, not timestamps, deliberately: the writing model reliably knows the current date but not a trustworthy clock or timezone, and a fabricated time corrupts ordering worse than a coarse date ([rationale](rationale.md#dates-not-timestamps-in-amr-lite)). Within one day, order is carried by the file itself — entries are appended, so at equal `rec` the lower entry is the later one; that file position is the conflict tie-break of the reading policy ([SPEC.md, section 5](SPEC.md#5-reading-policy-informative)).
