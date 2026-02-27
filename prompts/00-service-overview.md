# P0: Service Overview (Microservices)

> **Model:** Indexing (powerful model, one-time per service)
> **Purpose:** Generate 1-2 high-density chunks per service. Highest ROI prompt — run this first for ALL services before deep-diving any single service.

## Prompt

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

## Tips for Efficiency

- **Batch 2-3 small services per call** if each has a short README or single entry file
- **For services with no docs**, paste the main router/controller file — the model will infer purpose from routes
- **Run this for ALL services first** before doing any Phase 2-5 deep dives. The service map this produces is your most valuable retrieval asset
