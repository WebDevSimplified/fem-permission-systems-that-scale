---
title: "Converting to RBAC"
---

Let's convert our scattered permission checks into a centralized RBAC system.

## The Goal

Replace this scattered mess:

```typescript
// In page 1
if (user.role !== "admin" && user.role !== "author") {
  return redirect("/")
}

// In page 2 (slightly different!)
if (user.role === "viewer") {
  return redirect("/")
}

// In mutation
if (user == null || user.role === "viewer" || user.role === "editor") {
  throw new AuthorizationError()
}
```

With this centralized approach:

```typescript
// Everywhere
if (!can(user, "document:create")) {
  return redirect("/")
}
```

## Step 1: Create the Permissions Module

We'll create a new file that defines all permissions and the role mappings:

```typescript
// src/permissions/rbac.ts

import { User } from "@/drizzle/schema"

type Permission =
  | "project:create"
  | "project:read:all"
  | "project:read:own-department"
  | "project:read:global-department"
  | "project:update"
  | "project:delete"
  | "document:create"
  | "document:read"
  | "document:update"
  | "document:delete"

const permissionsByRole: Record<User["role"], Permission[]> = {
  admin: [
    "project:create",
    "project:read:all",
    "project:update",
    "project:delete",
    "document:create",
    "document:read",
    "document:update",
    "document:delete",
  ],
  author: [
    "project:read:own-department",
    "project:read:global-department",
    "document:create",
    "document:read",
    "document:update",
  ],
  editor: [
    "project:read:own-department",
    "project:read:global-department",
    "document:read",
    "document:update",
  ],
  viewer: [
    "project:read:own-department",
    "project:read:global-department",
    "document:read",
  ],
}

export function can(
  user: Pick<User, "role"> | null,
  permission: Permission,
): boolean {
  if (user == null) return false
  return permissionsByRole[user.role].includes(permission)
}
```

## Step 2: Handle Complex Permission Logic

Some permissions aren't purely role-based. For example, "can read project" depends on:

- The user's role (admins can read all)
- The user's department
- The project's department

We create helper functions for these:

```typescript
// src/permissions/projects.ts

import { Project, User } from "@/drizzle/schema"
import { can } from "./rbac"

export function canReadProject(
  user: Pick<User, "role" | "department"> | null,
  project: Pick<Project, "department">,
): boolean {
  if (user == null) return false

  return (
    can(user, "project:read:all") ||
    (can(user, "project:read:own-department") &&
      project.department === user.department) ||
    (can(user, "project:read:global-department") && project.department == null)
  )
}
```

Notice how we're already mixing RBAC (`can(user, "permission")`) with attribute checks (`project.department === user.department`). This is a hint of what's to come.

## Step 3: Update the Codebase

Now we replace all inline checks with our centralized functions.

### Before (Page Component)

```typescript
if (
  user == null ||
  (user.role !== "admin" &&
    project.department != null &&
    user.department !== project.department)
) {
  return redirect("/")
}
```

### After (Page Component)

```typescript
if (!canReadProject(user, project)) {
  return redirect("/")
}
```

### Before (Mutation)

```typescript
if (user == null || user.role === "viewer") {
  throw new AuthorizationError()
}
```

### After (Mutation)

```typescript
if (!can(user, "document:update")) {
  throw new AuthorizationError()
}
```

### Before (UI Conditional)

```typescript
{(user?.role === "author" || user?.role === "admin") && (
  <Button>New Document</Button>
)}
```

### After (UI Conditional)

```typescript
{can(user, "document:create") && (
  <Button>New Document</Button>
)}
```

## What We Gain

After this refactor:

| Benefit                    | Description                                           |
| -------------------------- | ----------------------------------------------------- |
| **Single source of truth** | All permissions defined in one place                  |
| **Readable code**          | `can(user, "document:create")` is self-documenting    |
| **Type safety**            | TypeScript ensures we only use valid permission names |
| **Easy updates**           | Change a role's permissions in one file               |

## Branch Checkpoint

After completing this conversion, your code should match:

```text
Branch: 3-basic-rbac
```

Run the following to sync up:

```bash
git checkout 3-basic-rbac
```

## What's Next

Our RBAC system works great for the current permissions. But what happens when we need to add more complex rules?
