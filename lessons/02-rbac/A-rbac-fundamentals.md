---
description: "Understand the core concepts of Role-Based Access Control: roles, permissions, and how they map together."
---

# RBAC Fundamentals

Role-Based Access Control (RBAC) is the most common and simplest permission system. If you've ever seen "Admin", "Editor", or "Viewer" in an application, you've seen RBAC.

## The Core Concept

In RBAC, permissions are assigned to **roles**, and users are assigned to **roles**. Users don't have permissions directly—they inherit them through their roles.

```text
User → Role → Permissions
```

## Why Roles?

Instead of assigning permissions to each user individually:

```text
❌ Direct assignment (hard to manage):
  Alice: [read, create, update]
  Bob: [read]
  Carol: [read, create, update]
  Dave: [read, create, update, delete]
```

We assign roles:

```text
✅ Role-based (easy to manage):
  Roles:
    viewer: [read]
    editor: [read, create, update]
    admin: [read, create, update, delete]

  Users:
    Alice → editor
    Bob → viewer
    Carol → editor
    Dave → admin
```

When we need to change what editors can do, we change it once in the role definition, not for every editor.

## Our Roles

For our Document Management System, we have three roles:

### Viewer

- Can read documents and comments
- Cannot create, edit, or delete anything

### Editor

- Can read all documents and comments
- Can create new documents
- Can update documents
- Cannot delete documents (for now—we'll add ownership later)

### Admin

- Can do everything
- Can manage users
- Can see sensitive fields

## Defining the Role-Permission Map

Here's how we'll define our RBAC rules in code:

```typescript
// permissions/roles.ts

export const ROLES = {
  viewer: ["read"],
  editor: ["read", "create", "update"],
  admin: ["read", "create", "update", "delete", "manage"],
} as const

export type Role = keyof typeof ROLES
export type Permission = (typeof ROLES)[Role][number]
```

## Checking Permissions

With this structure, checking if a user has a permission is straightforward:

```typescript
function hasPermission(userRole: Role, permission: Permission): boolean {
  return ROLES[userRole].includes(permission)
}

// Usage
hasPermission("editor", "read") // true
hasPermission("editor", "delete") // false
hasPermission("admin", "delete") // true
```

## Role Hierarchies (Optional Enhancement)

Some RBAC systems have **role inheritance**: admin inherits all editor permissions, editor inherits all viewer permissions.

```typescript
const ROLE_HIERARCHY = {
  admin: ["editor"], // admin inherits from editor
  editor: ["viewer"], // editor inherits from viewer
  viewer: [], // base role
}
```

We'll keep it simple for now without hierarchies, but this is a common pattern.

## RBAC Limitations Preview

RBAC alone can't express:

- "Editors can only edit **their own** documents" (ownership)
- "Users can only see documents in **their organization**" (multi-tenancy)
- "Published documents can't be edited" (resource state)
- "Can't edit on weekends" (environmental context)

We'll hit these limitations soon and solve them with ABAC.

## Next Steps

Let's implement this in our application:

1. Create the role-permission mapping
2. Add backend middleware to check roles
3. Update the frontend to show/hide UI elements

Let's write some code!
