---
title: "Choosing the Right Permission Model"
---

Let's recap what we've learned and when to use each approach.

## The Three Models

### RBAC (Role-Based Access Control)

Permissions are assigned to **roles**, and users are assigned to roles.

```typescript
const permissions = {
  admin: ["create", "read", "update", "delete"],
  editor: ["read", "update"],
  viewer: ["read"],
}

if (permissions[user.role].includes("update")) {
  // Allow action
}
```

**Use RBAC when:**

- Your permissions map cleanly to job functions
- You have a manageable number of roles
- Permissions don't depend on resource data

### ABAC (Attribute-Based Access Control)

Permissions are determined by **attributes** of the user, resource, action, and environment.

```typescript
const canUpdate =
  user.role === "editor" &&
  document.status !== "locked" &&
  user.department === document.department

if (canUpdate) {
  // Allow action
}
```

**Use ABAC when:**

- Permissions depend on resource properties
- You need fine-grained, contextual rules
- RBAC leads to too many roles

### ReBAC (Relationship-Based Access Control)

Permissions are determined by **relationships** between users and resources.

```typescript
// Can user view this document?
// Check: user → team → project → folder → document
const canView = await checkRelationship(user, "viewer", document)
```

**Use ReBAC when:**

- Users share resources with each other
- Permissions inherit through hierarchies
- Your app is collaboration-focused

## Quick Decision Guide

| Scenario                                | Recommended Model |
| --------------------------------------- | ----------------- |
| Simple app with admin/user roles        | RBAC              |
| Users can only edit their own content   | ABAC              |
| Permissions vary by department/location | ABAC              |
| Google Drive-style sharing              | ReBAC             |
| Folder → file permission inheritance    | ReBAC             |
| Enterprise with complex org structure   | ABAC or ReBAC     |

## They're Not Mutually Exclusive

Most real applications combine these models:

```typescript
// RBAC: Check role first
if (user.role === "admin") return true

// ABAC: Then check attributes
if (user.role === "editor" && !document.isLocked) return true

// ReBAC: Finally check relationships
if (await isSharedWith(user, document)) return true

return false
```

## Key Takeaways

1. **Start simple** - Begin with basic checks, refactor when needed
2. **Centralize authorization** - Use a permissions module or service layer
3. **Consider libraries** - CASL, Casbin, or managed services save time
4. **Test your permissions** - Authorization bugs are security bugs
5. **Think about queries** - Can you filter at the database level?

## What We Built

Throughout this course, we evolved from scattered permission checks to a clean, scalable system:

1. **Basic checks** → Scattered `if` statements
2. **RBAC** → Role-based permission objects
3. **ABAC** → Attribute conditions with a permission builder
4. **Services layer** → Centralized auth with DB query generation
5. **CASL** → Production-ready library

Each step solved real problems while introducing patterns that scale.

## Thank You!

You now have the knowledge to build permission systems that grow with your application. Whether you choose RBAC, ABAC, ReBAC, or a combination—you understand the tradeoffs and can make informed decisions.

Happy coding! 🎉
