---
description: "Brief overview of Google Zanzibar and production ReBAC systems like SpiceDB and OpenFGA."
---

# Google Zanzibar & Production ReBAC

For complex ReBAC at scale, purpose-built systems exist.

## Google Zanzibar

Google's internal authorization system handles permissions for Drive, YouTube, Cloud, etc. Key concepts:

- **Relation Tuples**: `(user:alice, viewer, doc:1)`
- **Userset Rewrites**: Define computed relationships
- **Zookies**: Consistent snapshots for permission checks

### Zanzibar Schema Example

```text
definition user {}

definition document {
  relation owner: user
  relation editor: user
  relation viewer: user

  permission view = viewer + editor + owner
  permission edit = editor + owner
  permission delete = owner
}
```

## Open Source Implementations

### SpiceDB (by AuthZed)

```bash
# Check permission
spicedb check document:1 view user:alice
```

### OpenFGA (by Auth0/Okta)

```typescript
const { allowed } = await fga.check({
  user: "user:alice",
  relation: "viewer",
  object: "document:1",
})
```

## When to Use

| Use Case            | Solution                    |
| ------------------- | --------------------------- |
| Simple sharing      | Custom code (what we built) |
| Complex hierarchies | SpiceDB / OpenFGA           |
| Massive scale       | Zanzibar-like system        |

## Key Takeaway

ReBAC is powerful but complex. Start with ABAC + simple sharing tables. Graduate to SpiceDB/OpenFGA when you outgrow that.

For this workshop, our custom implementation covers 90% of real-world needs.
