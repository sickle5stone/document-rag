# Document RAG

Structured knowledge retrieval for chat-based LLM environments.

Build a grounded, cited, searchable knowledge base from your documents — using nothing but markdown files and copy-paste prompts. No API keys, no vector databases, no infrastructure.

## When to Use This

- You have 10-100+ internal documents (specs, runbooks, code, policies)
- You need grounded, cited answers — not LLM guesswork
- You can't deploy RAG infrastructure (org constraints, no API access)
- You have access to LLM chat interfaces (any provider)
- Works for monoliths and microservices architectures alike

## Design

**Two-model strategy:**
- **Indexing model** (powerful, one-time per document): Extracts structured chunks with semantic summaries, keywords, and cross-references
- **Query model** (any tier, every question): Retrieves and answers using pre-structured chunks

All prompts are optimized so the query model can be significantly weaker than the indexing model.

## Quick Start (5 Minutes to First Answer)

```
1. Copy the prompt from prompts/01-intake-and-chunk.md
2. Paste one document into your indexing model with that prompt
3. Save the output to knowledge-base/domain/
4. Copy the prompt from prompts/06-chunk-selector.md
5. Paste the chunk titles + your question into your query model
6. Get a grounded, cited answer
```

## Repository Structure

```
document-rag/
├── README.md                  ← You are here
├── GUIDE.md                   ← Full methodology
├── PROMPTS.md                 ← All prompts in one file (copy-paste ready)
├── prompts/                   ← Individual prompt files
│   ├── 00-service-overview.md ← START HERE for microservices
│   ├── 01-intake-and-chunk.md
│   ├── 02-code-extraction.md
│   ├── 03-table-extraction.md
│   ├── 04-index-builder.md
│   ├── 05-cross-references.md
│   ├── 06-chunk-selector.md
│   ├── 07-grounded-query.md
│   ├── 08-maintenance-review.md
│   └── 09-cross-service-flow.md
├── templates/
│   ├── chunk.md               ← Chunk format template
│   └── index.md               ← Index format template
└── knowledge-base/
    ├── INDEX.md               ← Master index (start here for queries)
    ├── domain/                ← Domain knowledge chunks
    ├── codebase/              ← Code documentation chunks
    ├── processes/             ← Workflow and procedure chunks
    └── reference/             ← Standards, glossary, compliance chunks
```

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Semantic summary per chunk | Enables weak models to match query intent without embeddings |
| 300-800 token chunk limit | Fits 5 chunks + grounding prompt in an 8K context window |
| Combined intake + chunking | Saves 40-50% indexing compute by avoiding re-reading |
| 3-rule grounding prompt | Weak models follow short instructions more reliably |
| Category-prefixed IDs | Enables fast manual scanning without full-text search |
| Service-namespaced IDs | `[SVC]-[CAT]-[NNN]` keeps microservices organized |
| P0 service map first | 2 chunks/service gives 60% query coverage at minimal cost |

## Limitations

- Manual retrieval — you select chunks, not an algorithm
- Keyword + summary matching, not semantic embeddings
- Practical ceiling: ~100 documents before maintenance burden grows
- Designed as a bridge to proper RAG infrastructure, not a replacement

## Graduating to Full RAG

When your org approves infrastructure, your knowledge base exports cleanly:
- Chunks → vector DB documents (already properly sized and self-contained)
- INDEX.md → metadata store
- Keywords + summaries → hybrid search fields
- Cross-references → knowledge graph edges

## License

MIT
