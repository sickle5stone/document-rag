# P2: Code File Extraction

> **Model:** Indexing (powerful model, one-time per file)

## Prompt

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
