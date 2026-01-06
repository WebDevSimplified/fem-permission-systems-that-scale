---
description: "Implement the builder pattern for collecting permission rules."
---

# Builder Pattern

The builder collects rules via `allow()` and `deny()` calls.

## Live Code: createPermissions Function

```typescript
// packages/permissions/src/builder.ts

import { Rule, PermissionBuilder, PermissionContext } from "./types"
import { PermissionsImpl } from "./permissions"

export function createPermissions<TUser, TSubject extends string = string>(
  builder: PermissionBuilder<TUser, TSubject>,
  user: TUser
) {
  const rules: Rule<TSubject>[] = []

  // allow() can take conditions, fields, or both
  const allow: AllowFn<TSubject> = (
    action,
    subject,
    conditionsOrFields,
    fields
  ) => {
    const rule: Rule<TSubject> = {
      type: "allow",
      action,
      subject,
    }

    // Determine if second param is conditions or fields
    if (Array.isArray(conditionsOrFields)) {
      rule.fields = conditionsOrFields
    } else if (conditionsOrFields) {
      rule.conditions = conditionsOrFields
      if (fields) {
        rule.fields = fields
      }
    }

    rules.push(rule)
  }

  // deny() only takes conditions
  const deny: DenyFn<TSubject> = (action, subject, conditions) => {
    rules.push({
      type: "deny",
      action,
      subject,
      conditions,
    })
  }

  // Execute the builder
  const context: PermissionContext<TUser> = { user }
  builder(allow, deny, context)

  // Return the Permissions instance
  return new PermissionsImpl<TSubject>(rules)
}
```

## Usage Example

```typescript
const permissions = createPermissions<User, "Document" | "User">(
  (allow, deny, { user }) => {
    allow("read", "Document")
    allow("update", "Document", { authorId: user.id })
    allow("read", "Document", ["title", "content"])
    deny("update", "Document", { status: "archived" })
  },
  currentUser
)
```

Now let's implement the permission checking logic.
