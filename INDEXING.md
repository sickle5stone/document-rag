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

## Step 9: Deep Indexing — One Project at a Time

This step replaces the shallow Steps 3-5 with a thorough code-level analysis. Run it per-project so progress is saved even if you run out of tokens.

**How to use:** Paste this entire INDEXING.md into Opus (Copilot Agent Mode or chat), then say:

> `Deep index [project-name]. Follow Step 9 in INDEXING.md.`

To continue after a token break:

> `Continue deep indexing. Check knowledge-base/ for what's already done and pick up the next unfinished project.`

---

### 9a. Scope the project

1. Read `[project]/package.json` (or equivalent manifest). Extract: name, description, all scripts, all dependencies (prod + dev).
2. Map the full file tree: `[project]/src/**/*.{ts,js,py,go,java,rs}` — list every file with its path.
3. Classify each file into one of:
   - **CORE** — routes, controllers, handlers, services, models, middleware, jobs, workers, commands
   - **SUPPORT** — utils, helpers, validators, mappers, transformers, formatters
   - **TYPE** — types, interfaces, enums, constants, schemas (Zod/Joi/etc.)
   - **CONFIG** — database config, app config, env loading, dependency injection setup
   - **TEST** — test files, fixtures, mocks, factories
   - **SKIP** — generated files, index re-exports, barrel files with no logic

4. Report the file map:
```
[project-name] file map:
CORE:    [count] files — [list paths]
SUPPORT: [count] files — [list paths]
TYPE:    [count] files — [list paths]
CONFIG:  [count] files — [list paths]
TEST:    [count] files (skipping)
SKIP:    [count] files (skipping)
```

Read ALL files classified as CORE, SUPPORT, TYPE, and CONFIG before generating any chunks. Do not summarize from file names or imports alone.

---

### 9b. Generate the Architecture chunk

After reading all code, write one architecture chunk per project:

```markdown
## [Project Name] — Architecture

ID: [PREFIX]-ARCH-001
Keywords: [project name, architecture, structure, layers, patterns, 5-8 total]
Summary: [Describes how the project is structured internally and what patterns it uses]
Source: [project directory]

### Layer structure
[Describe the actual layers: e.g., routes → controllers → services → repositories → database]
[Name the specific files at each layer]

### Data flow
[Trace a typical request from entry point to response, naming actual functions and files]

### Key patterns
[List patterns found in the code: repository pattern, middleware chains, event emitters, dependency injection, etc.]
[For each pattern, cite the file(s) where it's implemented]

### State management
[How does this project manage state? Database, cache, in-memory, external service?]
[What ORMs, query builders, or database clients are used?]

### Error handling
[How are errors propagated? Custom error classes? Global error handler? Error codes?]

### Auth / security
[How does this project handle authentication and authorization?]
[Middleware, guards, decorators, API key validation — cite specific files]
```

---

### 9c. Generate Endpoint / Handler chunks

For each route file, controller, or command handler, generate a chunk:

```markdown
## [HTTP Method] [Route Path] — [Short Description]

ID: [PREFIX]-API-[NNN]
Keywords: [route path, handler name, HTTP method, domain terms, 5-8 total]
Summary: [What does this endpoint do and when would you call it?]
Source: [file path:line number]

### Request
- Method: [GET/POST/PUT/DELETE/etc.]
- Path: [exact route path with params]
- Auth: [required/optional/none — what auth middleware is applied]
- Body/Query/Params: [list each field with type and validation rules from the actual code]

### Logic (trace the full execution path)
[Step-by-step what the handler does — follow every function call to its implementation:]
1. [Validates X using Y function in Z file — list every field checked and rejection conditions]
2. [Calls serviceA.method(args) → describe what that method does internally, not just that it's called]
3. [Inside serviceA.method(): queries table X WHERE condition, transforms result by doing Y, then...]
4. [If condition A → path 1 (describe), else → path 2 (describe)]
5. [Writes to database table X with fields {a, b, c} / publishes event Y with payload {x, y}]
6. [Returns response shaped like {field: type, ...}]

DO NOT stop at "calls the service." Follow the call into the service, into the repository, down to the database query. The goal is that someone reading this chunk understands the EXACT logic without opening the source code.

### Response
- Success: [status code, response shape with actual field names]
- Errors: [list each error case with status code and when it triggers]

### Side effects
[Database writes, events emitted, external API calls, cache invalidation, emails sent — be specific]

### Dependencies
[List every service, repository, external client this handler calls — with file paths]
```

---

### 9d. Generate Service / Business Logic chunks

For each service file or significant business logic module:

```markdown
## [Service/Module Name] — [Short Description]

ID: [PREFIX]-SVC-[NNN]
Keywords: [service name, domain concepts, operations, 5-8 total]
Summary: [What business capability does this service provide?]
Source: [file path]

### Public interface
[List every exported function/method with signature and one-line description]

### Internal logic
[For each non-trivial function, write pseudocode-level detail:]
- Step-by-step what it does (reference actual variable names and conditions)
- Business rules enforced (e.g., "rejects orders over $10k without manager approval — line 47")
- Database queries: what table, what WHERE clause, what JOINs (describe intent + shape)
- External service calls: what method, what payload, what it does with the response
- Conditional branches: "if X → does Y, else → does Z" (capture the actual branching logic)
- Loops and iterations: what collection is iterated, what happens per item

### Data transformations
[What data does this service reshape? Input format → output format]

### Edge cases handled
[List specific edge cases the code handles: null checks, empty arrays, race conditions, retries]

### Error scenarios
[What errors can this service throw? Under what conditions?]
```

---

### 9e. Generate Model / Schema chunks

For each database model, schema definition, or significant type:

```markdown
## [Model/Type Name] — Schema

ID: [PREFIX]-MDL-[NNN]
Keywords: [model name, table name, entity, fields, 5-8 total]
Summary: [What entity does this represent and what data does it hold?]
Source: [file path]

### Fields
| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
[List every field from the actual schema/model definition]

### Relationships
[Foreign keys, references, has-many/belongs-to — name the related models]

### Indexes
[List any indexes defined in the code or migration files]

### Validation rules
[Zod schemas, Joi validators, class-validator decorators — list actual constraints]

### Used by
[Which services/controllers read or write this model? List file paths]
```

---

### 9f. Generate Config / Integration chunks

For each significant config file, external integration, or infrastructure setup:

```markdown
## [Config/Integration Name]

ID: [PREFIX]-CFG-[NNN]
Keywords: [config name, service name, environment, 5-8 total]
Summary: [What does this configure and what happens if it's wrong?]
Source: [file path]

### Settings
[List every config value with its purpose — preserve actual defaults, ports, URLs, timeouts]

### Environment variables required
| Variable | Purpose | Default | Required |
|----------|---------|---------|----------|
[List from .env.example, config loading code, or docker-compose]

### External connections
[Databases, caches, message brokers, third-party APIs — list connection details from code]
```

---

### 9g. Generate Middleware / Cross-cutting chunks

For auth middleware, logging, rate limiting, validation middleware, error handlers:

```markdown
## [Middleware Name] — [Short Description]

ID: [PREFIX]-MID-[NNN]
Keywords: [middleware name, purpose, where applied, 5-8 total]
Summary: [What does this middleware do and which routes use it?]
Source: [file path]

### Behavior
[What this middleware does to the request/response cycle — step by step]

### Applied to
[List specific routes or route groups that use this middleware]

### Configuration
[Any options, thresholds, or toggles this middleware accepts]

### Failure behavior
[What happens when this middleware rejects a request? Status code, error format]
```

---

### 9h. Write and checkpoint

1. Write ALL chunks for the project to: `document-rag/knowledge-base/codebase/[project-name].md` (one file per project, all chunk types together)
2. Write the overview + dependency chunks to: `document-rag/knowledge-base/domain/[project-name].md` (same as Step 3 but richer)
3. Write config chunks to: `document-rag/knowledge-base/reference/[project-name]-config.md`
4. After each project completes, update `document-rag/knowledge-base/INDEX.md` with all new chunk IDs

**Checkpoint report after each project:**
```
✓ [project-name] deep indexed
  Chunks written: [total count]
  - ARCH: [count]  API: [count]  SVC: [count]
  - MDL: [count]   CFG: [count]  MID: [count]
  Files: codebase/[project-name].md, domain/[project-name].md
  Next project: [name] or "All done"
```

---

### 9i. Quality rules for deep indexing

- **Read the code, don't guess.** Every claim in a chunk must come from a line of code you read. If you can't read a file, say "unread" — don't infer.
- **Name names.** Use actual function names, variable names, file paths, table names. "It calls a service" is worthless. "It calls `OrderService.calculateTotal()` in `src/services/order.service.ts:45`" is useful.
- **Trace the full call chain.** For endpoints, follow the logic from route → controller → service → repository → database. Don't stop at "calls the service layer."
- **Capture business rules, not syntax.** "Validates the input" tells me nothing. "Rejects if quantity < 1 or total exceeds user's credit limit (checked via `CreditService.check()`)" tells me everything.
- **Capture exact logic, not summaries.** The goal is that someone can understand implementation details from chunks alone without reading source. Write conditional branches (`if X → Y, else → Z`), loop behavior, transformation steps. Think "detailed pseudocode with real names," not "high-level overview."
- **Follow calls to their conclusion.** When a handler calls a service, describe what that service does. When the service calls a repository, describe the query. Don't leave any black boxes.
- **Include what's NOT handled.** If there's no input validation, no error handling, no auth check — note it. Gaps are as valuable as features.
- **Increase chunk size limit for deep indexing.** Chunks in `codebase/` may go up to 1500 tokens if needed to capture full logic. Prefer one complete chunk over two fragments that lose context.
- **One project at a time.** Finish writing all chunks and updating INDEX.md for one project before starting the next. This is the resume point if tokens run out.
- **Skip secrets.** Never include actual API keys, passwords, connection strings, or tokens in chunks. Describe what they connect to, not the values.

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
