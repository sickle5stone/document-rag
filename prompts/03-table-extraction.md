# P3: Table and Diagram Extraction

> **Model:** Indexing (powerful model, one-time)

## Prompt

```
Convert this table/diagram into knowledge chunks.

Rules:
1. Create ONE summary chunk describing the table's purpose and structure.
2. Create individual chunks ONLY for rows with unique or critical information.
3. Preserve ALL specific values: numbers, dates, thresholds, configurations.
4. Target 300-800 tokens per chunk.

Format each chunk as:

---
## [Descriptive Title]
ID: [CAT]-[NNN]
Keywords: [5-8 keywords]
Summary: [What question does this chunk answer?]
Source: [document name], [table/figure reference]

[Content in natural language, not table format]
---

TABLE/DIAGRAM:
[paste or describe the content here]
```
