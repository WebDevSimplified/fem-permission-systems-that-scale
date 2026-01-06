---
description: "Introduction to CASL, a popular isomorphic authorization library for JavaScript/TypeScript."
---

# CASL Overview

CASL (pronounced "castle") is the most popular JavaScript authorization library. It uses a **no-DSL approach** - you define permissions in pure JavaScript/TypeScript.

## Philosophy

- **Isomorphic**: Same code works on frontend and backend
- **No DSL**: Pure JavaScript, no config files
- **Framework integrations**: React, Vue, Angular adapters
- **Database integration**: Generate Prisma/Mongoose queries from abilities

## Core Concepts

### Ability

The central class holding all permission rules:

```typescript
import { defineAbility } from "@casl/ability"

const ability = defineAbility((can, cannot) => {
  can("read", "Document")
  can("update", "Document", { authorId: userId })
  cannot("delete", "Document")
})
```

### Actions

Verbs like `read`, `create`, `update`, `delete`, or custom like `publish`, `share`.

### Subjects

Resources being accessed - strings (`'Document'`) or class instances.

### Conditions

MongoDB-style queries restricting when rules apply:

```typescript
can("update", "Document", { authorId: user.id })
can("read", "Document", { status: { $in: ["draft", "published"] } })
```

## Checking Permissions

```typescript
ability.can("read", "Document") // Type check
ability.can("update", someDocumentInstance) // Instance check
ability.cannot("delete", "Document") // Inverse

if (ability.can("update", document)) {
  // Allowed
}
```

Let's refactor our app to use CASL.
