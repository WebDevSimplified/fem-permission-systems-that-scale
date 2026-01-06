---
description: "Add TypeScript types to CASL for compile-time safety and IDE autocomplete."
---

# CASL TypeScript Integration

CASL has excellent TypeScript support. Let's add strict types.

## Live Code: Type Definitions

```typescript
// server/src/permissions/ability.ts

import {
  createMongoAbility,
  MongoAbility,
  AbilityBuilder,
  InferSubjects,
} from "@casl/ability"

// Define our domain types
interface Document {
  __typename: "Document"
  id: number
  authorId: number
  orgId: number
  status: string
}

interface User {
  __typename: "User"
  id: number
  role: string
  orgId: number
}

// Define actions
type Actions = "create" | "read" | "update" | "delete" | "manage"

// Define subjects (can be string or instance)
type Subjects = InferSubjects<Document | User> | "all"

// Create typed ability
export type AppAbility = MongoAbility<[Actions, Subjects]>

export function defineAbilityFor(user: {
  id: number
  role: string
  orgId: number
}) {
  const { can, cannot, build } = new AbilityBuilder<AppAbility>(
    createMongoAbility
  )

  can("read", "Document", { orgId: user.orgId })

  if (user.role === "admin") {
    can("manage", "all")
  }

  if (user.role === "editor") {
    can("create", "Document")
    can("update", "Document", { authorId: user.id })
  }

  return build()
}
```

## Type Safety Benefits

```typescript
const ability = defineAbilityFor(user)

// ✅ TypeScript knows these are valid
ability.can("read", "Document")
ability.can("update", "User")

// ❌ TypeScript error: invalid action
ability.can("fly", "Document")

// ❌ TypeScript error: invalid subject
ability.can("read", "Spaceship")
```

## IDE Autocomplete

With proper types, your IDE will suggest:

- Valid actions when you type `ability.can('`
- Valid subjects after the action
- Available condition fields based on subject type

This catches permission bugs at compile time!
