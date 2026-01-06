---
description: "Generate Drizzle ORM where clauses from permission rules."
---

# Drizzle Integration

Let's generate database filters from our permission rules.

## Live Code: filter Method

```typescript
// packages/permissions/src/drizzle.ts

import { eq, and, or, SQL } from "drizzle-orm"
import { Rule } from "./types"

export function buildDrizzleFilter(
  rules: Rule[],
  action: string,
  subject: string,
  table: Record<string, any>
): SQL | undefined {
  // Get all allow rules for this action/subject
  const allowRules = rules.filter(
    (r) =>
      r.type === "allow" &&
      (r.action === action || r.action === "manage") &&
      (r.subject === subject || r.subject === "all")
  )

  if (allowRules.length === 0) {
    // No rules = deny all, return impossible condition
    return eq(table.id, -1) // Nothing will match
  }

  // Build conditions for each rule
  const conditions: SQL[] = []

  for (const rule of allowRules) {
    if (!rule.conditions) {
      // No conditions = allow all for this rule
      return undefined // No filter needed
    }

    // Build AND conditions for this rule
    const ruleConditions: SQL[] = []

    for (const [field, value] of Object.entries(rule.conditions)) {
      if (!(field in table)) continue

      if (Array.isArray(value)) {
        // OR for array values
        const orConditions = value.map((v) => eq(table[field], v))
        ruleConditions.push(or(...orConditions)!)
      } else {
        ruleConditions.push(eq(table[field], value))
      }
    }

    if (ruleConditions.length > 0) {
      conditions.push(and(...ruleConditions)!)
    }
  }

  // Combine all rule conditions with OR
  if (conditions.length === 0) return undefined
  if (conditions.length === 1) return conditions[0]
  return or(...conditions)
}
```

## Add to PermissionsImpl

```typescript
filter(action: string, subject: TSubject, table: Record<string, any>): SQL | undefined {
  return buildDrizzleFilter(this.rules, action, subject, table);
}
```

## Usage

```typescript
// Define permissions
const permissions = createPermissions((allow, deny, { user }) => {
  allow("read", "Document", { orgId: user.orgId })
  if (user.role === "admin") {
    allow("read", "Document") // Admin sees all in org
  }
}, currentUser)

// Generate filter
const filter = permissions.filter("read", "Document", documentsTable)

// Use in query
const docs = await db.select().from(documentsTable).where(filter)
```

This ensures users only get data they're allowed to see!
