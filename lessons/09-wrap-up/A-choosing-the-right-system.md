---
description: "A decision framework for choosing between RBAC, ABAC, and ReBAC based on your application's needs."
---

# Choosing the Right System

Use this framework to decide which permission approach fits your needs.

## Decision Flowchart

```text
Start
  │
  ▼
Do users need to access only THEIR OWN resources?
  │
  ├─ No → Simple RBAC is probably fine
  │
  ▼ Yes
  │
Do you have multi-tenant requirements?
  │
  ├─ No → ABAC with ownership checks
  │
  ▼ Yes
  │
Do users need to SHARE resources with each other?
  │
  ├─ No → ABAC with org scoping
  │
  ▼ Yes
  │
Is sharing simple (direct user → resource)?
  │
  ├─ Yes → ABAC + share table
  │
  ▼ No (nested: org → team → resource)
  │
Consider ReBAC or dedicated system (SpiceDB, OpenFGA)
```

## Quick Reference

| If you need...             | Use                  |
| -------------------------- | -------------------- |
| Basic role protection      | RBAC                 |
| Ownership ("my documents") | ABAC                 |
| Org isolation              | ABAC + org scoping   |
| Field-level control        | ABAC + field rules   |
| Direct sharing             | ABAC + share table   |
| Complex hierarchies        | ReBAC                |
| Massive scale sharing      | Zanzibar-like system |

## Hybrid is Normal

Most real apps use multiple approaches:

```text
RBAC for: Route-level protection ("admins can access /admin")
ABAC for: Resource-level rules ("edit own documents")
ReBAC for: Sharing features ("shared with me")
```

Don't overthink it. Start simple, add complexity only when needed.
