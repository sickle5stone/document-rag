# Indexing Instructions for Claude Code

Run Claude Code (Opus) from the workspace root directory and say:
**"Follow document-rag/INDEXING.md to index my codebase"**

---

## Step 1: Discovery

Scan the workspace root directory.

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

## Step 3: Index Each Project (P0 — Service Overview)

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

Using the content you read, generate exactly 2 chunks:

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

## Step 4: Index Root-Level Docs (P1 — Document Intake)

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

## Step 5: Build INDEX.md

After all chunks are generated, read every `.md` file in:
- `document-rag/knowledge-base/domain/`
- `document-rag/knowledge-base/codebase/`
- `document-rag/knowledge-base/processes/`
- `document-rag/knowledge-base/reference/`

Extract each chunk's ID, Title, Summary, and Keywords.

Write `document-rag/knowledge-base/INDEX.md` with this structure:

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
(etc.)

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

## Step 6: Deep Indexing (Optional — On Request Only)

Only run this when the user asks to go deeper on a specific project.

1. Glob for code files in the target project: `[project]/src/**/*.{ts,js,py,go,java}`
2. Focus on: routes, controllers, handlers, services, models — skip tests, utils, types/interfaces
3. Read each significant file
4. Generate P2 chunks (code extraction):

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

## Rules

- **Do NOT index files containing secrets** (.env, credentials, API keys). If you encounter one, skip it.
- **Do NOT include company names, product names, or identifying information** in chunk content. Use generic descriptions.
- **Keep chunks self-contained.** Each chunk must be understandable alone.
- **Keep chunks under 800 tokens.** The query model has limited context.
- **Summaries describe what QUESTION the chunk answers**, not what it contains.
  - Bad: "Contains information about the auth service"
  - Good: "Describes how the auth service validates tokens and handles session expiry"
- **Keywords include synonyms and abbreviations.** If a chunk is about "distributed ledger", also add: "DLT", "blockchain", "ledger".
- **Process projects in alphabetical order** for reproducibility.
- **Report progress** after each project: "[N/total] Indexed [dirname] → [PREFIX]-SYS-001, [PREFIX]-SYS-002"
