# Document RAG — Complete Reference

This file is self-contained. Paste it into Opus as your instructions, then paste your project files for processing. All templates, query prompts, and maintenance workflows are in this one document.

## How to Use

**First run — Opus via Copilot Agent Mode (indexing):**
1. Open workspace root in VS Code (all project folders + document-rag/)
2. Select **Opus** as the model in Copilot Chat
3. Enable **Agent Mode** (so Copilot can read and write files)
4. Say: `@workspace Follow document-rag/INDEXING.md to index my codebase. Start with Step 1.`
5. Copilot will scan projects, generate chunks, and write them to `knowledge-base/`
6. Process in batches if context gets long: `Continue with the next 3 projects.`
7. After all projects: `Run Step 8 — build INDEX.md.`

**Ongoing queries — cheaper model:**
1. Switch to a faster/cheaper model in Copilot Chat
2. Use the query prompts from Part 2 (Chunk Selector → Grounded Query)

**Adding new projects later:**
1. Clone new repo into workspace root
2. Same prompt — Opus reads INDEX.md, sees it's not in the Source Map, indexes only the new project

---

# Knowledge Base Folder Structure

Create this structure before starting. All chunks and the index go here.

```
document-rag/knowledge-base/
├── INDEX.md               ← Master index (generated in Step 8)
├── domain/                ← Service overviews and domain knowledge
│   ├── [project-a].md    ← One file per project (Steps 3)
│   └── [project-b].md
├── codebase/              ← Code-level documentation (Step 9, optional)
│   └── [project]-[file].md
├── processes/             ← Workflows and procedures
│   └── cross-service-flows.md  ← End-to-end flows (Step 6)
└── reference/             ← Root-level docs, configs, standards
    ├── [root-doc].md      ← Root .md files (Step 4)
    └── [project]-config.md ← Config extractions (Step 5)
```

---

# Part 1: Indexing

---

## Step 1: Discovery

**If running in Claude Code:** Scan the workspace root directory automatically.
**If pasting into Opus chat:** List your project folders and root docs below, or paste them one at a time.

**Detect projects:** A directory is a project if it contains any of: `.git/`, `package.json`, `Dockerfile`, `go.mod`, `Cargo.toml`, `requirements.txt`, `pom.xml`.

**Skip these directories:**
`.github`, `.idea`, `.venv`, `.vscode`, `node_modules`, `archive`, `working_directory`, `document-rag`, `dist`, `build`, `coverage`, `__pycache__`

**Collect root-level docs:** `.md` files in the root directory (not in subdirectories).

**Skip root-level files:** `.env`, `*.ps1`, `.gitignore`, `package.json`, `*.lock`, `*.log`

**Check what's already indexed:** If `knowledge-base/INDEX.md` has a Source Map section, extract the list of already-indexed items. Skip those — only process new ones.

Report:
```
Projects detected: [count]
Already indexed:   [count]
New to index:      [count]
Root docs to index: [count]
```

**Detect projects:** List immediate subdirectories. A directory is a project if it contains any of: `.git/`, `package.json`, `Dockerfile`, `go.mod`, `Cargo.toml`, `requirements.txt`, `pom.xml`.

**Skip these directories:**
`.github`, `.idea`, `.venv`, `.vscode`, `node_modules`, `archive`, `working_directory`, `document-rag`, `dist`, `build`, `coverage`, `__pycache__`

**Collect root-level docs:** Find `.md` files in the root directory (not in subdirectories).

**Skip root-level files:** `.env`, `*.ps1`, `.gitignore`, `package.json`, `*.lock`, `*.log`

**Check what's already indexed:** Read `document-rag/knowledge-base/INDEX.md`. If it has a Source Map section, extract the list of already-indexed projects and root docs. Skip those — only process new/unindexed items.

Report what you found:
```
Projects detected: [count]
Already indexed:   [count]
New to index:      [count]
Root docs to index: [count]
```

---

## Step 2: Generate Service Prefixes

For each project directory, generate a short uppercase prefix (2-6 chars):

1. Take the directory name
2. Strip common suffixes: `-service`, `-api`, `-app`, `-server`, `-ui`, `-infra`, `-console`, `-contract`, `-volume`
3. If the remainder is ≤4 characters, uppercase it (e.g., `fx` → `FX`)
4. If longer, take first letter of each hyphen-separated word (e.g., `data-pipeline` → `DP`, `user-portal` → `UP`)
5. If the result is only 1 character, use the first 3 characters of the original name
6. If there's a collision with an existing prefix, append a number

For root-level docs, use prefix `PLATFORM`.

---

## Step 3: Index Each Project — Service Overview

For each new project, in order:

### 3a. Find the best source file

Read ONE of these (in priority order, first one that exists):
1. `[project]/README.md`
2. `[project]/docs/README.md`
3. `[project]/package.json` (extract name, description, scripts, dependencies)
4. Main entry: `[project]/src/index.ts`, `[project]/src/app.ts`, `[project]/index.js`, `[project]/main.go`, `[project]/app.py`
5. Main router: first file matching `[project]/src/routes/**/*.ts` or `[project]/src/controllers/**/*.ts`

If none exist, skip the project and note it as "no indexable entry point".

### 3b. Generate 2 chunks

**Chunk 1: Overview**
```markdown
## [Project Name] — Overview

ID: [PREFIX]-SYS-001
Keywords: [project name, aliases, domain terms, technologies, 5-8 total]
Summary: [What this project does and why it exists — one sentence that answers "what is this?"]
Source: [file you read]

Purpose: [2-3 sentences: what business capability this provides]
Tech stack: [language, framework, database, message broker if detectable]
Owns: [what data or domain concepts this project is the source of truth for]
Key endpoints/exports: [top 3-5 endpoints, commands, or exports with one-line descriptions]
```

**Chunk 2: Dependencies**
```markdown
## [Project Name] — Dependencies

ID: [PREFIX]-SYS-002
Keywords: [project name, dependencies, integration, upstream, downstream]
Summary: [What this project depends on and what depends on it — one sentence]
Source: [file you read]

Depends on: [list each: service/system → what it provides]
Depended on by: [list each: service/system → what it consumes. Write "Unknown — needs investigation" if not clear]
Communication: [sync (REST/gRPC) vs async (events/queues) for each dependency]
Failure impact: [what breaks if this project goes down]
```

### 3c. Write the chunks

Write both chunks to: `document-rag/knowledge-base/domain/[dirname].md`

If the file already exists, append the new chunks (don't overwrite).

---

## Step 4: Index Root-Level Docs — Document Intake

For each unindexed root `.md` file:

### 4a. Read the file

### 4b. Generate chunks

Extract self-contained knowledge chunks:
- One topic per chunk
- Each chunk understandable WITHOUT reading other chunks
- Preserve specifics: numbers, dates, versions, configs, thresholds
- Keep multi-step procedures in one chunk
- Target 300-800 tokens per chunk
- 5-8 keywords per chunk including synonyms and abbreviations
- One-sentence Summary that states what QUESTION this chunk answers

Format:
```markdown
## [Descriptive Title]

ID: PLATFORM-[CAT]-[NNN]
Keywords: [keyword1, keyword2, keyword3, keyword4, keyword5]
Summary: [e.g. "Describes how X works" or "Lists the steps to Y"]
Source: [filename]

[Extracted content — facts only, no interpretation]
```

Categories: SYS=Systems, PROC=Processes, ARCH=Architecture, API=Endpoints, CFG=Configuration, SEC=Security, OPS=Operations, REF=Reference

### 4c. Write the chunks

Write to: `document-rag/knowledge-base/reference/[filename].md`

---

## Step 5: Index Config and Structured Data

For each project, check for config files that contain significant domain information:
- `docker-compose.yml`, `Dockerfile`
- `*.yaml`, `*.yml` (OpenAPI specs, deployment configs)
- Environment config templates (`.env.example`, `config.ts`)

Only index configs that reveal architecture, not boilerplate.

Format:
```markdown
## [Config/Resource Name]

ID: [PREFIX]-CFG-[NNN]
Keywords: [5-8 keywords]
Summary: [What question does this chunk answer?]
Source: [file path]

[Configuration described in natural language. Preserve all specific values: ports, URLs, thresholds, feature flags.]
```

Write to: `document-rag/knowledge-base/reference/[project]-config.md`

---

## Step 6: Detect Cross-Service Flows

After indexing all projects, review the dependency chunks (all *-SYS-002 chunks). Identify workflows that span 3+ services.

For each detected flow:

```markdown
## [Flow Name] — End-to-End

ID: PLATFORM-PROC-[NNN]
Keywords: [flow name, services involved, trigger, outcome, 5-8 total]
Summary: [What business process this flow accomplishes, one sentence]
Source: [inferred from dependency chunks]

Trigger: [What initiates this flow]
Steps:
1. [Service A] → [action] → [output/event]
2. [Service B] → [action] → [output/event]
3. ...

Happy path result: [What happens when everything works]
Failure modes:
- [Step N fails] → [consequence] → [recovery/retry behavior]

Services involved: [list all service prefixes]
Related chunks: [PREFIX-SYS-001, PREFIX-SYS-002, etc.]
```

Write to: `document-rag/knowledge-base/processes/cross-service-flows.md`

---

## Step 7: Generate Cross-References

Review all chunks. For each chunk, identify strong relationships:
- Describes parts of the same system
- Upstream/downstream dependency
- Same topic from different angles
- One references a concept the other explains

Add a `Related:` line to each chunk that has 2+ strong relationships.

---

## Step 8: Build INDEX.md

Read every `.md` file in:
- `document-rag/knowledge-base/domain/`
- `document-rag/knowledge-base/codebase/`
- `document-rag/knowledge-base/processes/`
- `document-rag/knowledge-base/reference/`

Extract each chunk's ID, Title, Summary, and Keywords.

Write `document-rag/knowledge-base/INDEX.md`:

```markdown
# Knowledge Base Index

Last updated: [today's date]
Total chunks: [count]

---

## By Service

### [Service Prefix] — [Project Name]
| ID | Title | Summary |
|----|-------|---------|
| [ID] | [Title] | [Summary] |

(Repeat for each project. Group PLATFORM chunks separately.)

---

## By Category

### Systems (SYS)
| ID | Title | Summary |
|----|-------|---------|
(All SYS chunks across all services)

### Processes (PROC)
(etc. for each category that has chunks)

---

## By Topic

### [Keyword appearing in 2+ chunks]
[comma-separated chunk IDs]

(Generate 10-20 topic groups)

---

## Source Map

| Project / Document | Prefix | Chunks | Date Indexed |
|-------------------|--------|--------|-------------|
| [dirname or filename] | [PREFIX] | [ID range] | [today] |
```

---

## Step 9: Deep Indexing (Optional — On Request Only)

Only run when the user asks to go deeper on a specific project.

1. Glob for code files: `[project]/src/**/*.{ts,js,py,go,java}`
2. Focus on: routes, controllers, handlers, services, models — skip tests, utils, types/interfaces
3. Read each significant file
4. Generate code extraction chunks:

```markdown
## [Function/Class/Module Name]

ID: [PREFIX]-API-[NNN]
Keywords: [function name, what it does, technologies, related terms]
Summary: [What question does this chunk answer?]
Source: [file path]
Type: [function / class / module / config / endpoint]

Purpose: [1-2 sentences, plain English]
Inputs: [parameters, types, expected values]
Outputs: [return values, side effects]
Dependencies: [other functions/services/systems called]
Key logic: [important business logic in plain English]
Edge cases: [how failures are handled]
```

5. Write to `document-rag/knowledge-base/codebase/[project]-[filename].md`
6. Update INDEX.md

---

# Part 2: Querying (Manual — Use in Any Chat Interface)

Copy these prompts into your query model (Copilot, ChatGPT, etc.).

---

## Chunk Selector — Find Relevant Chunks

Use this when you have a question and need to find which chunks to read.

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

## Grounded Query — Get a Cited Answer

Use this after you have the relevant chunks.

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

# Part 3: Maintenance

---

## Knowledge Base Review (Quarterly)

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

## Adding New Projects

When you clone a new repo into the workspace root:
1. Re-run: "Follow document-rag/INDEXING.md to index my codebase"
2. Opus reads INDEX.md, sees the new project isn't in the Source Map
3. Only the new project gets indexed
4. INDEX.md gets updated with the new entries

## Updating Existing Chunks

When information changes:
- **Replace:** Update chunk content, change the date. Use when old info is fully superseded.
- **Version:** Keep both chunks. Add `Supersedes: [OLD-ID]` to the new one.
- **Deprecate:** Mark with `Status: DEPRECATED`. Remove from INDEX.md but keep the file.

---

# Rules

- **Do NOT index files containing secrets** (.env, credentials, API keys). Skip them.
- **Do NOT include company names, product names, or identifying information** in chunks. Use generic descriptions.
- **Keep chunks self-contained.** Each chunk must be understandable alone.
- **Keep chunks under 800 tokens.** The query model has limited context.
- **Summaries describe what QUESTION the chunk answers**, not what it contains.
  - Bad: "Contains information about the auth service"
  - Good: "Describes how the auth service validates tokens and handles session expiry"
- **Keywords include synonyms and abbreviations.** If a chunk is about "distributed ledger", also add: "DLT", "blockchain", "ledger".
- **Process projects in alphabetical order** for reproducibility.
- **Report progress** after each project: "[N/total] Indexed [dirname] → [PREFIX]-SYS-001, [PREFIX]-SYS-002"
