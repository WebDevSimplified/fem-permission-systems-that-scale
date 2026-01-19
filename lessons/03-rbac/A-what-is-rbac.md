---
title: "What is RBAC?"
---

**Role-Based Access Control (RBAC)** is the most common permission system you'll encounter. If you've ever assigned a user to an "admin" or "editor" role, you've used RBAC.

## The Core Concept

RBAC is built on a simple idea: **permissions are assigned to roles, not users**. Users are then assigned roles, and they inherit all permissions associated with those roles.

```text
User → Role → Permissions
```

Instead of asking "Can this user do X?", you ask "Does this user's role have permission X?"

## The RBAC Model

A typical RBAC implementation has three components:

| Component       | Description                            | Example                         |
| --------------- | -------------------------------------- | ------------------------------- |
| **Users**       | The people using your system           | Alice, Bob                      |
| **Roles**       | Named groups of permissions            | admin, editor, viewer           |
| **Permissions** | Specific actions that can be performed | create:document, delete:project |

## How Permissions Flow

```text
┌─────────┐     ┌─────────┐     ┌──────────────────┐
│  Alice  │─────│  admin  │─────│ create:document  │
└─────────┘     └─────────┘     │ delete:document  │
                                │ create:project   │
                                │ delete:project   │
                                └──────────────────┘

┌─────────┐     ┌─────────┐     ┌──────────────────┐
│   Bob   │─────│ viewer  │─────│ read:document    │
└─────────┘     └─────────┘     └──────────────────┘
```

Alice (admin) can create and delete anything. Bob (viewer) can only read documents. Neither user has permissions defined directly—they inherit from their roles.

## Why RBAC Works

RBAC became popular for good reasons:

### 1. Conceptually Simple

Everyone understands roles. "Make Bob an admin" is clearer than "Grant Bob create:document, delete:document, update:document, create:project..."

### 2. Easy to Manage

When permissions change, you update the role once. All users with that role automatically get the new permissions.

### 3. Auditable

"Who has delete access?" → "Everyone with the admin role" → Easy to audit.

### 4. Scales with Users

Adding a new user? Assign them a role. Done. No need to configure individual permissions.

## RBAC in Code

A basic RBAC implementation looks like this:

```typescript
// Define your permissions
type Permission =
  | "document:create"
  | "document:read"
  | "document:update"
  | "document:delete"
  | "project:create"
  | "project:read"
  | "project:update"
  | "project:delete"

// Map roles to permissions
const permissionsByRole: Record<string, Permission[]> = {
  admin: [
    "document:create",
    "document:read",
    "document:update",
    "document:delete",
    "project:create",
    "project:read",
    "project:update",
    "project:delete",
  ],
  editor: ["document:read", "document:update"],
  viewer: ["document:read"],
}

// Check if a user can perform an action
function can(user: { role: string }, permission: Permission): boolean {
  return permissionsByRole[user.role]?.includes(permission) ?? false
}
```

Usage is clean and readable:

```typescript
if (can(user, "document:create")) {
  // Show create button
}

if (!can(user, "project:delete")) {
  throw new AuthorizationError()
}
```

## Role Hierarchies (Optional Enhancement)

Some RBAC implementations include **role hierarchies**, where roles inherit from other roles:

```text
admin → author → editor → viewer
```

In this model:

- `admin` has all permissions of `author` (plus admin-specific ones)
- `author` has all permissions of `editor` (plus author-specific ones)
- `editor` has all permissions of `viewer` (plus editor-specific ones)

This may seem convenient, but it drastically reduces the flexibility of your RBAC system and usually doesn't scale well.

## When RBAC Shines

RBAC is ideal when:

- Permissions are **role-based** (not resource-specific)
- You have a **small, fixed set of roles**
- Permission logic is **straightforward** (no complex conditions)
- You need something **quick to implement** and **easy to understand**

## When RBAC Struggles

RBAC starts to break down when:

- Permissions depend on **resource attributes** (e.g., "can edit documents they created")
- Permissions depend on **relationships** (e.g., "can edit documents in projects they own")
- You have **many roles** with slight permission variations
- Business rules are **complex and context-dependent**

We'll experience these limitations firsthand later in this section.
