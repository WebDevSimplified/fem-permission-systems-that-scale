---
title: "What is ABAC?"
---

**Attribute-Based Access Control (ABAC)** is a permission system that makes decisions based on **attributes** of users, resources, and the environment. Instead of asking "Does this role have permission?", ABAC asks "Do the attributes of this request satisfy the policy?"

## The Core Concept

ABAC evaluates permissions using four categories of attributes:

| Category        | Description                 | Examples                                                     |
| --------------- | --------------------------- | ------------------------------------------------------------ |
| **Subject**     | Who is making the request   | `user.role`, `user.department`, `user.id`                    |
| **Resource**    | What is being accessed      | `document.status`, `document.creatorId`, `document.isLocked` |
| **Action**      | What operation is requested | `read`, `create`, `update`, `delete`                         |
| **Environment** | Contextual factors          | Time of day, IP address, device type                         |

## How ABAC Differs from RBAC

```text
RBAC:  User → Role → Permission
ABAC:  (Subject + Resource + Action + Environment) → Policy → Decision
```

In RBAC, you ask: "Does the admin role have the document:update permission?"

In ABAC, you ask: "Can a user with `role=editor` and `department=Engineering` update a document with `status=draft` and `isLocked=false`?"

## A Simple ABAC Policy

Here's what an ABAC policy looks like conceptually:

```typescript
// "Authors can edit their own unlocked draft documents"
{
  action: "update",
  resource: "document",
  conditions: {
    "user.role": "author",
    "document.creatorId": "user.id",  // Dynamic comparison!
    "document.isLocked": false,
    "document.status": "draft"
  }
}
```

## Why ABAC Solves Our Problems

Remember the permissions that broke our RBAC system?

| Requirement          | RBAC Approach                                               | ABAC Approach                             |
| -------------------- | ----------------------------------------------------------- | ----------------------------------------- |
| "Edit own documents" | Create `document:update:own` permission + manual check      | `condition: { creatorId: user.id }`       |
| "Edit unlocked only" | Create `document:update:unlocked` permission + manual check | `condition: { isLocked: false }`          |
| "View non-drafts"    | Create `document:read:non-draft` permission + manual check  | `condition: { status: { $ne: "draft" } }` |

ABAC expresses these naturally as **conditions**, not encoded permission strings.

## ABAC in Code

Here's a simplified ABAC implementation:

```typescript
function getUserPermissions(user: User) {
  const builder = new PermissionBuilder()

  if (user.role === "author") {
    builder
      .allow("document", "read", { status: "published" })
      .allow("document", "read", { status: "archived" })
      .allow("document", "read", { status: "draft", creatorId: user.id })
      .allow("document", "create")
      .allow("document", "update", {
        creatorId: user.id,
        isLocked: false,
        status: "draft",
      })
  }

  return builder.build()
}

// Usage
const permissions = getUserPermissions(user)
permissions.can("document", "update", document) // Checks all conditions!
```

## When ABAC Shines

ABAC is ideal when:

- Permissions depend on **resource attributes** (status, ownership, flags)
- Permissions depend on **relationships** (creator, team member, project owner)
- You need **fine-grained control** over specific fields
- Business rules are **complex and context-dependent**
- You want a **single, consistent API** for all permission checks

## When ABAC Adds Complexity

ABAC isn't free—it comes with tradeoffs:

- **More complex to reason about** than simple role checks
- **Policies can become intricate** with many conditions
- **Performance considerations** when checking many conditions
- **Database queries** may need to incorporate permission filters
- **Type safety** requires careful design

## Our Implementation Plan

In the upcoming lessons, we'll:

1. **Convert** our RBAC system to a basic ABAC system (same functionality, better architecture)
2. **Add advanced features** like field-level permissions
3. **Create a services layer** that centralizes all authorization logic
4. **Convert permissions to database queries** for efficient filtering

Let's start by building a clean ABAC foundation.
