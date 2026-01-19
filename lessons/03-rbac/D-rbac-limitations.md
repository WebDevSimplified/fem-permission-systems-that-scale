---
title: "RBAC Limitations"
---

Let's summarize when RBAC works well and when you should consider alternatives.

## Where RBAC Excels

RBAC is the right choice when your permissions are:

| Characteristic                  | Example                                                        |
| ------------------------------- | -------------------------------------------------------------- |
| **Role-centric**                | "Admins can delete, viewers can read"                          |
| **Static**                      | Permissions don't change based on context                      |
| **Coarse-grained**              | A few roles cover all use cases                                |
| **Limited Attribute Awareness** | Permissions depend on few or no resource attributes or context |

## Where RBAC Struggles

RBAC breaks down when permissions depend on:

### 1. Attributes Other Than User Role

As soon as you need to create multiple permutations of permissions for different attributes (e.g., ownership, document status), RBAC starts to struggle. You end up creating very specific permissions like `"document:update:own-unlocked-draft"` to cover each combination which quickly becomes unmanageable.

### 5. Environmental Factors

With RBAC it is impossible to take into consideration environmental factors such as time of day, location, or other contextual information that may play a role in determining access. For example, a user may only be able to push code to production during certain hours or from specific IP addresses.

## The Symptom: Permission Explosion

When RBAC is the wrong fit, you'll notice your permission list growing uncontrollably:

```typescript
// Started simple
type Permission = "document:read" | "document:update" | "document:delete"

// Then requirements came in...
type Permission =
  | "document:read:all"
  | "document:read:own"
  | "document:read:non-draft"
  | "document:read:published"
  | "document:update:all"
  | "document:update:unlocked"
  | "document:update:own"
  | "document:update:own-unlocked"
  | "document:update:own-unlocked-draft"
  | "document:delete:all"
  | "document:delete:own"
// ... and it keeps growing
```

## The Hybrid Trap

When teams hit RBAC limits, they often create hybrids:

```typescript
// "RBAC" that's actually policy functions
export function canUpdateDocument(user, document) {
  if (user.role === "admin") return !document.isLocked
  if (user.role === "editor") return !document.isLocked
  if (user.role === "author") {
    return (
      document.creatorId === user.id &&
      !document.isLocked &&
      document.status === "draft"
    )
  }
  return false
}
```

This works, but:

- You've abandoned the simplicity of RBAC
- Permission logic is scattered across helper functions
- The `can(user, permission)` pattern is inconsistent
- You're essentially hand-building ABAC without the structure

## When to Graduate from RBAC

Consider moving beyond RBAC when you find yourself:

1. **Creating permission names with multiple conditions**
   - `"document:update:own-unlocked-draft"` is a red flag

2. **Writing helper functions for every permission check**
   - If `canDoX(user, resource)` is the norm, RBAC isn't helping

3. **Duplicating attribute checks across the codebase**
   - The same `document.isLocked` check appearing everywhere

4. **Struggling to explain permissions to new team members**
   - "Well, editors can update documents, except when..."

5. **Needing different permissions for the same action based on context**
   - Same "edit" action has different rules for drafts vs published

## The Path Forward

In the next section, we'll implement **ABAC** and see how it elegantly handles the requirements that broke our RBAC system. ABAC is more work to set up initially, but it scales much better for complex, attribute-driven access control scenarios.
