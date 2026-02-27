# P4: Index Builder

> **Model:** Indexing (one-time, after all documents are processed)

## Prompt

```
Build a master index from these knowledge chunks.

Output format:

# Knowledge Base Index
Last updated: [today's date]
Total chunks: [count]

## By Category

### [Category Name] ([PREFIX])
| ID | Title | Summary |
|----|-------|---------|
| [ID] | [Title] | [Summary line from chunk] |

(Repeat for each category that has chunks)

## By Topic

Group chunk IDs under common themes:
### [Theme name]
[comma-separated chunk IDs]

(Create 5-15 topic groups based on content overlap)

## Source Map
| Source Document | Chunks |
|----------------|--------|
| [doc name] | [ID range] |

CHUNKS:
[paste all chunks here]
```
