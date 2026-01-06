---
description: "Load permission rules from the database and build abilities dynamically."
---

# Dynamic Permission Loading

Now let's load rules from the database instead of hardcoding them.

## Live Code: Load Rules from DB

```typescript
// server/src/permissions/dynamicPermissions.ts

import { db } from "../db"
import { permissionRulesTable } from "../db/schema"
import { eq } from "drizzle-orm"
import { createPermissions } from "@our-app/permissions"

interface User {
  id: number
  role: string
  orgId: number
}

export async function createDynamicPermissions(user: User) {
  // Fetch rules for this user's role
  const rules = await db
    .select()
    .from(permissionRulesTable)
    .where(eq(permissionRulesTable.role, user.role))

  return createPermissions((allow, deny, { user: u }) => {
    for (const rule of rules) {
      // Resolve special placeholders in conditions
      const conditions = resolveConditions(rule.conditions, u)

      if (rule.type === "allow") {
        if (rule.fields) {
          allow(rule.action, rule.subject, conditions, rule.fields)
        } else if (conditions) {
          allow(rule.action, rule.subject, conditions)
        } else {
          allow(rule.action, rule.subject)
        }
      } else {
        deny(rule.action, rule.subject, conditions)
      }
    }
  }, user)
}

function resolveConditions(
  conditions: Record<string, any> | null,
  user: User
): Record<string, any> | undefined {
  if (!conditions) return undefined

  const resolved: Record<string, any> = {}

  for (const [key, value] of Object.entries(conditions)) {
    if (value === "$user.id") {
      resolved[key] = user.id
    } else if (value === "$user.orgId") {
      resolved[key] = user.orgId
    } else {
      resolved[key] = value
    }
  }

  return resolved
}
```

## Usage in Routes

```typescript
router.get("/", requireAuth, async (req, res) => {
  // Dynamically loaded permissions!
  const permissions = await createDynamicPermissions(req.user!)

  const docs = await db
    .select()
    .from(documentsTable)
    .where(permissions.filter("read", "Document", documentsTable))

  res.json(docs.map((d) => permissions.filterFields(d)))
})
```

Now permission changes take effect immediately without redeployment!
