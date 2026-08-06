# Examples

| File | What it shows |
|---|---|
| [record.json](record.json) | a full record (AMR-Extended: all MUST and SHOULD fields) |
| [record.md](record.md) | the same record as Markdown front matter, plus its AMR-lite line |
| [supersession-chain.json](supersession-chain.json) | a fact changing over time: v1 marked superseded, v2 active, linked both ways |

Content hashes are real SHA-256 digests of the `content` field; recompute with `printf '%s' "<content>" | sha256sum`.

## Validating against the schema

All JSON examples validate against [../schema/amr.schema.json](../schema/amr.schema.json) (JSON Schema draft 2020-12). With [check-jsonschema](https://check-jsonschema.readthedocs.io/) (`pip install check-jsonschema`):

```sh
check-jsonschema --schemafile ../schema/amr.schema.json record.json
```

`supersession-chain.json` is an array of records, so validate each element:

```sh
jq -c '.[]' supersession-chain.json | while read -r record; do
  echo "$record" | check-jsonschema --schemafile ../schema/amr.schema.json -
done
```
