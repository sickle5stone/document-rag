# P1: Document Intake + Chunk Extraction

> **Model:** Indexing (powerful model, one-time per document)

## Prompt

```
You are a knowledge base indexer. Read the document and produce two outputs.

=== OUTPUT 1: DOCUMENT SUMMARY ===

- Title: [document title]
- Topics: [5-10 main topics, comma-separated]
- Entities: [systems, teams, products, technologies mentioned]
- Key terms: [term: one-line definition] for each domain-specific term

=== OUTPUT 2: KNOWLEDGE CHUNKS ===

Extract self-contained chunks. Rules:
1. One topic per chunk. Each chunk must be understandable WITHOUT other chunks.
2. Preserve specifics: numbers, dates, versions, configs, thresholds, error codes.
3. Keep multi-step procedures in one chunk.
4. Target 300-800 tokens per chunk. Split longer content.
5. Write 5-8 keywords per chunk including synonyms and abbreviations.
6. Write a one-sentence Summary that states what QUESTION this chunk answers.

Format each chunk exactly as:

---
## [Descriptive Title]
ID: [CAT]-[NNN]
Keywords: [keyword1, keyword2, keyword3, keyword4, keyword5]
Summary: [e.g. "Describes how X works" or "Lists the steps to Y"]
Source: [document name], [section/page]

[Extracted content — facts only, no interpretation]
---

Categories:
SYS=Systems, PROC=Processes, ARCH=Architecture, API=Endpoints,
CFG=Configuration, SEC=Security, OPS=Operations, REF=Reference

DOCUMENT:
[paste document here]
```
