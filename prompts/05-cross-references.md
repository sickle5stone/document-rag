# P5: Cross-Reference Generator

> **Model:** Indexing (after indexing, or when adding new documents)

## Prompt

```
Review these chunk summaries and identify cross-references.

For each chunk, list other chunks that are related because they:
- Describe parts of the same system
- Are upstream/downstream dependencies
- Cover the same topic from different angles
- One references a concept the other explains

Output format:

[CHUNK-ID] → Related: [ID1, ID2, ID3] — [one sentence explaining why]

Only include strong relationships. Skip weak or obvious ones.

CHUNK SUMMARIES:
[paste INDEX.md or list of ID + Summary lines]
```
