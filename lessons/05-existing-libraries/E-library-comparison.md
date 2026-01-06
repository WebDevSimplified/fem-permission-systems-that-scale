---
description: "Compare CASL and Casbin to understand when to use each library."
---

# Library Comparison

Let's compare the two approaches side by side.

## Same Rules, Different Syntax

### CASL (Code-based)

```typescript
const ability = defineAbility((can) => {
  can("read", "Document", { orgId: user.orgId })
  can("update", "Document", { authorId: user.id })

  if (user.role === "admin") {
    can("manage", "Document")
  }
})

// Check
ability.can("update", document)
```

### Casbin (DSL-based)

```ini
# model.conf
[matchers]
m = r.sub.orgId == r.obj.orgId && \
    (r.sub.role == "admin" || r.sub.id == r.obj.authorId)
```

```csv
# policy.csv
p, editor, document, read
p, editor, document, update
p, admin, document, *
```

```typescript
const allowed = await enforcer.enforce(
  { id: user.id, role: user.role, orgId: user.orgId },
  { authorId: doc.authorId, orgId: doc.orgId },
  "update"
)
```

## Decision Criteria

### Choose CASL when:

- ✅ Pure JavaScript/TypeScript project
- ✅ Want strong type safety
- ✅ Developers manage permissions
- ✅ Need database query generation
- ✅ Want React/Vue components

### Choose Casbin when:

- ✅ Multi-language microservices
- ✅ Need external policy management
- ✅ Complex RBAC with domains
- ✅ Want to separate policy from code
- ✅ Need policy versioning/auditing

## Our Choice

For this workshop, we'll build something similar to CASL because:

1. Pure TypeScript - great DX
2. No external files to manage
3. Easy to understand internals
4. We can customize it exactly how we want

Let's build our own permission library!
