# Document RAG — Full Methodology

A step-by-step guide for building a structured, retrievable knowledge base using chat-based LLM interfaces.

---

## Prerequisites

- Access to a powerful LLM via chat interface (for one-time indexing)
- Access to any LLM for ongoing queries (can be a weaker/cheaper model)
- Your source documents (specs, code, PDFs, runbooks, policies)
- A place to store markdown files (local folder, Git repo, wiki)

---

## Phase 1: Setup

### Step 1: Create the folder structure

```
knowledge-base/
├── INDEX.md
├── domain/
├── codebase/
├── processes/
└── reference/
```

### Step 2: Understand the chunk format

Every chunk follows this format (see `templates/chunk.md`):

```markdown
## [Descriptive Title]

ID: [CAT]-[NNN]
Keywords: keyword1, keyword2, keyword3, keyword4, keyword5
Summary: One sentence describing what question this chunk answers
Source: [document name], [section/page]
Related: [IDs of related chunks]

[Content — facts only, self-contained, 300-800 tokens]
```

**Why this format matters for weak query models:**
- **Summary** line enables semantic matching without embeddings — the query model reads summaries to find relevant chunks
- **Keywords** include synonyms and abbreviations for lexical matching
- **300-800 token limit** ensures 5 chunks fit in an 8K context window with room for the grounding prompt
- **Self-contained** means each chunk works alone — no need to fetch dependencies

### Step 3: Understand the two-model strategy

| Phase | Model | Runs | Optimize for |
|-------|-------|------|-------------|
| Indexing | Powerful (one-time) | Once per document | Richness — extract maximum useful metadata |
| Querying | Any tier (ongoing) | Every question | Simplicity — follow short instructions reliably |

The indexing model does the heavy lifting once. The query model just follows simple retrieval and grounding instructions.

---

## Phase 2: Indexing (One-Time Per Document)

Use your most capable model for this phase. All prompts are in `PROMPTS.md`.

### Step 4: Process a document

Use **Prompt P1** (`prompts/01-intake-and-chunk.md`).

This is a combined intake + chunking prompt that produces both a document summary and structured chunks in a single pass. This saves ~40-50% compute versus running two separate passes.

**Save the output:**
- Document summary → append to `knowledge-base/INDEX.md`
- Chunks → save to appropriate folder (`domain/`, `codebase/`, `processes/`, or `reference/`)

### Step 5: Process code files

Use **Prompt P2** (`prompts/02-code-extraction.md`).

Optimized for code — extracts purpose, inputs, outputs, key logic, and edge cases. Skips trivial getters/setters.

### Step 6: Process tables and diagrams

Use **Prompt P3** (`prompts/03-table-extraction.md`).

Converts structured data into natural language chunks. One summary chunk for the table, plus individual chunks only for rows with unique or critical information.

### Step 7: Handle large documents

If a document exceeds the context window:

```
1. Split at natural section boundaries
2. Process each section with P1, adding this header:
   "Section [X] of [Y] from [document name].
    Previous sections covered: [topic list]."
3. After all sections, use P5 to generate cross-references
```

### Step 8: Saving compute on the indexing model

Strategies for limited compute budgets:

1. **Batch small documents** — Combine 2-3 short documents (under 2K tokens each) in a single P1 call
2. **Skip boilerplate** — Remove table of contents, headers-only pages, and legal disclaimers before pasting
3. **Prioritize** — Index your most-queried documents first. Add others only when needed
4. **One pass only** — The combined P1 prompt eliminates the need for separate summary and chunking passes

---

## Phase 3: Organize

### Step 9: Review IDs and keywords

After extraction, review each chunk:

1. **Verify the ID** follows `[CAT]-[NNN]` format with these categories:
   ```
   SYS  — Systems and platforms
   PROC — Processes and workflows
   ARCH — Architecture and design decisions
   API  — API endpoints and integrations
   CFG  — Configuration and environment
   SEC  — Security and access control
   OPS  — Operations and monitoring
   REF  — Standards, compliance, reference
   ```

2. **Check keywords** — Ask yourself: "In 3 months, what words would I search for?" Add those. Include synonyms and abbreviations.

3. **Check the summary** — It should describe what QUESTION this chunk answers, not what it contains.
   - Bad: "Contains information about the auth service"
   - Good: "Describes how the auth service validates JWT tokens and handles expiry"

### Step 10: Build the index

Use **Prompt P4** (`prompts/04-index-builder.md`).

Paste all your chunks and the model generates INDEX.md. Or build it manually using `templates/index.md`.

### Step 11: Add cross-references

Use **Prompt P5** (`prompts/05-cross-references.md`).

Paste your INDEX.md and the model identifies which chunks should reference each other.

---

## Phase 4: Querying

Use any model for this phase. Prompts are designed for weaker/cheaper models.

### Step 12: Find relevant chunks

**Option A — You know which chunks to use:**
Open INDEX.md, scan by category or keyword, note the chunk IDs.

**Option B — You're not sure which chunks to use:**
Use **Prompt P6** (`prompts/06-chunk-selector.md`).
Paste your INDEX.md and your question. The model suggests 3-5 relevant chunk IDs based on summaries and keywords.

### Step 13: Get a grounded answer

Use **Prompt P7** (`prompts/07-grounded-query.md`).

Paste the selected chunks and your question. The model answers using ONLY the chunk content, with inline citations.

**This prompt is intentionally short (3 rules).** Weak models follow short instructions more reliably than long ones.

### Step 14: The complete query workflow

```
Question
  → Paste INDEX.md + question into P6 (chunk selector)
  → Get suggested chunk IDs
  → Copy those chunks from your knowledge base files
  → Paste chunks + question into P7 (grounded query)
  → Get cited, grounded answer
```

Two copy-paste operations. That's it.

---

## Phase 5: Maintenance

### Step 15: Adding new documents

```
1. Process with P1 (or P2/P3 for code/tables)
2. Save chunks to appropriate folder
3. Add entries to INDEX.md
4. Run P5 on new chunks + existing INDEX.md for cross-references
```

### Step 16: Updating existing information

When information changes:

- **Replace:** Update chunk content, change the date. Use when old info is fully superseded.
- **Version:** Keep both chunks if both versions might be relevant. Add: `Supersedes: [OLD-ID]`
- **Deprecate:** Mark with `Status: DEPRECATED`. Remove from INDEX.md but keep the file.

### Step 17: Periodic review

Use **Prompt P8** (`prompts/08-maintenance-review.md`) quarterly.

Paste INDEX.md to identify gaps, overlaps, stale entries, and missing cross-references.

**Do not over-maintain.** If maintenance takes more than 10 minutes per week, you're doing too much. Only update chunks when the underlying information actually changes.

---

## Comparison: This System vs Full RAG

| Capability | Document RAG | Full RAG Pipeline |
|------------|-------------|-------------------|
| Grounded, cited answers | Yes | Yes |
| Structured knowledge | Yes (manual) | Yes (automated) |
| Semantic matching | Partial (summary-based) | Yes (embedding-based) |
| 10-50 documents | Well | Well |
| 100+ documents | Strained | Well |
| Automatic retrieval | No | Yes |
| Context quality | High (human judgment) | Variable (algorithm) |
| Setup cost | Time only | Infrastructure + engineering |
| Maintenance | Manual updates | Automated re-indexing |
| Export to full RAG | Clean (chunks are ready) | N/A |

This system works well up to ~100 documents. Beyond that, export your knowledge base into a proper RAG pipeline — the structured chunks you've built become the ideal corpus.
