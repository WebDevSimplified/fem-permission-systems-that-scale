---
description: "Refactor our permission system to use CASL's defineAbility and ability checks."
---

# Implementing CASL

Let's replace our custom permission code with CASL.

## Installation

```bash
npm install @casl/ability
```

## Live Code: Define Abilities

```typescript
// server/src/permissions/ability.ts

import {
  defineAbility,
  AbilityBuilder,
  createMongoAbility,
} from "@casl/ability"

interface User {
  id: number
  role: "viewer" | "editor" | "admin"
  orgId: number
}

export function defineAbilityFor(user: User) {
  return defineAbility((can, cannot) => {
    // Everyone can read documents in their org
    can("read", "Document", { orgId: user.orgId })

    if (user.role === "viewer") {
      // Viewers can only read
      return
    }

    if (user.role === "editor") {
      // Editors can create documents
      can("create", "Document")

      // Editors can update their own documents
      can("update", "Document", { authorId: user.id, orgId: user.orgId })

      // But not published documents
      cannot("update", "Document", { status: "published" })
    }

    if (user.role === "admin") {
      // Admins can do everything in their org
      can("manage", "Document", { orgId: user.orgId })
      can("manage", "User", { orgId: user.orgId })
    }
  })
}
```

## Live Code: Using in Routes

```typescript
// server/src/routes/documents.ts

import { defineAbilityFor } from "../permissions/ability"
import { subject } from "@casl/ability"

router.put("/:id", requireAuth, async (req, res) => {
  const ability = defineAbilityFor(req.user!)
  const document = await getDocument(req.params.id)

  // CASL check - pass the actual document instance
  if (ability.cannot("update", subject("Document", document))) {
    return res.status(403).json({ error: "Forbidden" })
  }

  // Proceed with update...
})
```

## The `subject` Helper

CASL needs to know the subject type when checking instances:

```typescript
import { subject } from "@casl/ability"

// ❌ Won't work - CASL doesn't know it's a Document
ability.can("update", document)

// ✅ Works - explicitly typed
ability.can("update", subject("Document", document))
```

Or add a `__typename` property to your objects.
