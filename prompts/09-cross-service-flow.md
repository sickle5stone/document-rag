# P9: Cross-Service Flow Extraction

> **Model:** Indexing (powerful model, one-time per flow)
> **Purpose:** Document end-to-end workflows that span multiple services

## Prompt

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

## When to Use

- User authentication and token refresh flows
- Payment/transaction processing pipelines
- Data synchronization between services
- Event-driven workflows (order placed → inventory reserved → notification sent)
- Any "how does X work end-to-end" question that touches 3+ services
