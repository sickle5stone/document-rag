# Document RAG — All Prompts

Copy-paste ready. Replace `[PLACEHOLDERS]` with your content.

**Service mapping (P0):** Run FIRST for all services before deep-diving.
**Indexing prompts (P1-P5):** Use with your most capable model. Run once per document.
**Query prompts (P6-P7):** Use with any model (optimized for weaker/cheaper models). Run per question.
**Maintenance prompts (P8):** Run quarterly.
**Cross-service flows (P9):** Use for end-to-end workflows spanning multiple services.

ID format for microservices: `[SERVICE]-[CAT]-[NNN]` (e.g., `AUTH-API-001`, `PAY-PROC-003`)
Use `PLATFORM` prefix for cross-cutting concerns.

---

## P0: Service Overview (Microservices)

> **Use with:** Indexing model (powerful, one-time per service)
> **Purpose:** 1-2 high-density chunks per service. Highest ROI — run this first for ALL services.

```
You are a knowledge base indexer for a microservices platform.

For EACH service described below, produce exactly ONE overview chunk and ONE dependency chunk.

=== OVERVIEW CHUNK FORMAT ===

---
## [Service Name] — Overview
ID: [SVC]-SYS-001
Keywords: [service name, aliases, domain terms, 5-8 total]
Summary: [What this service does and why it exists, one sentence]
Source: [source file/doc]

Purpose: [2-3 sentences: what business capability this service provides]
Tech stack: [language, framework, database, message broker]
Owns: [what data/domain concepts this service is the source of truth for]
Key endpoints: [list top 3-5 endpoints with one-line descriptions]
---

=== DEPENDENCY CHUNK FORMAT ===

---
## [Service Name] — Dependencies
ID: [SVC]-SYS-002
Keywords: [service name, dependencies, integration, upstream, downstream]
Summary: [What this service depends on and what depends on it]
Source: [source file/doc]

Depends on: [list: service/system → what it provides]
Depended on by: [list: service/system → what it consumes]
Communication: [sync (REST/gRPC) vs async (events/queues), list each]
Failure impact: [what breaks if this service goes down]
---

Use short service-code prefixes (2-4 chars): AUTH, PAY, GW, USR, etc.
Keep each chunk under 500 tokens. Facts only.

SERVICE INFORMATION:
[paste service README, main router/controller, or architecture doc here]
```

---

## P1: Document Intake + Chunk Extraction

> **Use with:** Indexing model (powerful, one-time per document)
> **Purpose:** Combined summary + chunked extraction in a single pass

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

---

## P2: Code File Extraction

> **Use with:** Indexing model (powerful, one-time per file)
> **Purpose:** Extract documentation chunks from source code

```
Analyze this code and extract documentation chunks.

For each significant function, class, or module create a chunk.
Skip trivial getters, setters, and boilerplate.
Target 300-800 tokens per chunk.

Format each chunk as:

---
## [Function/Class/Module Name]
ID: API-[NNN]
Keywords: [function name, what it does, technologies, related terms]
Summary: [One sentence: what question does this chunk answer?]
Source: [file path]
Type: [function / class / module / config / endpoint]

Purpose: [1-2 sentences, plain English]
Inputs: [parameters, types, expected values]
Outputs: [return values, side effects]
Dependencies: [other functions/services/systems called]
Key logic: [important business logic explained in plain English]
Edge cases: [how failures are handled]
---

CODE:
[paste code here]
```

---

## P3: Table and Diagram Extraction

> **Use with:** Indexing model (powerful, one-time)
> **Purpose:** Convert structured data into natural language chunks

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

---

## P4: Index Builder

> **Use with:** Indexing model (one-time, after all documents are processed)
> **Purpose:** Generate INDEX.md from all chunks

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

---

## P5: Cross-Reference Generator

> **Use with:** Indexing model (after indexing, or when adding new documents)
> **Purpose:** Identify relationships between chunks

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

---

## P6: Chunk Selector

> **Use with:** Query model (any tier — optimized for weaker models)
> **Purpose:** Find relevant chunks for a question

```
You are a retrieval assistant.

Given the INDEX below, return the 3-5 chunk IDs most relevant to the QUESTION.

For each ID, write one sentence explaining why it is relevant.

INDEX:
[paste INDEX.md here]

QUESTION:
[your question here]
```

---

## P7: Grounded Query

> **Use with:** Query model (any tier — optimized for weaker models)
> **Purpose:** Answer a question using only provided chunks

```
Answer the question using ONLY the chunks below.

Rules:
1. Use ONLY information from the chunks. If not found, say "Not in provided chunks."
2. Cite chunk IDs inline like this: [SYS-001]
3. If chunks conflict, note both versions with their IDs.

CHUNKS:
[paste selected chunks here]

QUESTION:
[your question here]
```

---

## P8: Maintenance Review

> **Use with:** Either model (quarterly)
> **Purpose:** Identify gaps, overlaps, and stale entries

```
Review this knowledge base index. Identify:

1. GAPS: Topics that seem incomplete or missing
2. OVERLAPS: Chunks that may duplicate information
3. STALE: Chunks that may need updating (check dates)
4. MISSING LINKS: Chunks that should cross-reference each other but don't

For each finding, cite the specific chunk IDs involved.

INDEX:
[paste INDEX.md here]
```

---

## P9: Cross-Service Flow Extraction

> **Use with:** Indexing model (powerful, one-time per flow)
> **Purpose:** Document end-to-end workflows spanning multiple services

```
Document this end-to-end workflow that spans multiple services.

Produce ONE chunk per workflow. Keep under 800 tokens.

Format:

---
## [Flow Name] — End-to-End
ID: PLATFORM-PROC-[NNN]
Keywords: [flow name, services involved, trigger, outcome, 5-8 total]
Summary: [What business process this flow accomplishes, one sentence]
Source: [architecture doc, sequence diagram, or code files]

Trigger: [What initiates this flow]
Steps:
1. [Service A] → [action] → [output/event]
2. [Service B] → [action] → [output/event]
3. ...

Happy path result: [What happens when everything works]
Failure modes:
- [Step N fails] → [consequence] → [recovery/retry behavior]

Services involved: [list all service prefixes]
Related chunks: [SVC-SYS-001, SVC-SYS-002, etc.]
---

WORKFLOW INFORMATION:
[paste sequence diagram, architecture doc, or describe the flow]
```

---

## Prompt Cheat Sheet

```
MICROSERVICES — START HERE:
  Each service → P0 (service overview) → 2 chunks/service → INDEX.md
  Cross-service flows → P9 (flow extraction) → PLATFORM-PROC chunks

INDEXING A DOCUMENT:
  Document → P1 (intake + chunk) → save chunks + update INDEX.md
  Code     → P2 (code extraction) → save chunks + update INDEX.md
  Table    → P3 (table extraction) → save chunks + update INDEX.md
  All docs → P4 (build index) → generate INDEX.md
  New docs → P5 (cross-references) → update Related fields

ANSWERING A QUESTION:
  Question → P6 (chunk selector) with INDEX.md → get chunk IDs
           → P7 (grounded query) with chunks → get cited answer

MAINTAINING:
  Quarterly → P8 (review) with INDEX.md → fix gaps/overlaps/stale
```
