# P7: Grounded Query

> **Model:** Query model (any tier — optimized for weaker models)

## Prompt

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
