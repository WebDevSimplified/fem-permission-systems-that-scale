---
description: "Design the API for our custom permission library with a focus on simplicity and type safety."
---

# API Design

Before writing code, let's design our API. We want something simpler than CASL but just as powerful for our needs.

## Design Goals

1. **Simple** - Easy to understand internals
2. **Type-safe** - Catch errors at compile time
3. **Equality only** - No complex operators (keeps it simple)
4. **Field-level** - Built-in support for field permissions
5. **Drizzle-ready** - Can generate database filters

## Proposed API

### Defining Permissions

```typescript
const permissions = createPermissions<User>((allow, deny, { user }) => {
  // Role-based
  if (user.role === "admin") {
    allow("manage", "Document")
  }

  // With conditions (equality only)
  allow("update", "Document", { authorId: user.id })

  // Field-level
  allow("read", "Document", ["title", "content"])
  allow("read", "Document", ["internalNotes"], { role: "admin" })

  // Explicit deny (overrides allow)
  deny("update", "Document", { status: "archived" })
})
```

### Checking Permissions

```typescript
permissions.can("read", document) // boolean
permissions.can("update", document) // boolean
permissions.can("read", document, "authorId") // field check

permissions.cannot("delete", document) // inverse
permissions.authorize("update", document) // throws if denied
```

### Field Helpers

```typescript
permissions.allowedFields("read", "Document") // ['title', 'content', ...]
permissions.allowedFields("write", "Document") // ['title', 'content']
permissions.filterFields(document) // strips unauthorized fields
```

### Database Filters

```typescript
permissions.filter("read", "Document") // Returns Drizzle where clause
```

## Key Differences from CASL

| CASL                             | Ours                      |
| -------------------------------- | ------------------------- |
| MongoDB operators (`$in`, `$gt`) | Equality only             |
| `subject()` helper needed        | Automatic type detection  |
| Separate field abilities         | Built into main `allow()` |
| External adapters                | Built-in Drizzle support  |

Let's build it!
