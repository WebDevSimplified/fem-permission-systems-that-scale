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

## How RBAC Works

Before we write code, let's visualize what we're building:

![RBAC Flow](/fem-permission-systems-that-scale/images/03-rbac/rbac-flow.svg)

The role acts as an indirection layer between users and permissions. Instead of scattering role checks everywhere, we create a **lookup table** that maps roles to permissions, then check against that table.

## Step 1: Create the Permissions Module

We'll create a new file `src/permissions/rbac.ts` and build it piece by piece.

### Part 1: Define the Permission Type

First, we define all possible permissions as a TypeScript union type:

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
```

Using a union type means TypeScript will catch typos like `"documnet:create"` at compile time—a huge win over string literals scattered throughout your codebase.

### Part 2: Map Roles to Permissions

Next, we create the lookup table—this is the heart of RBAC:

```typescript
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
```

Now when requirements change—say, authors should be able to delete their own documents—you update **one place** instead of hunting through dozens of files.

### Part 3: The `can` Function

Finally, a simple function to check permissions:

```typescript
export function can(
  user: Pick<User, "role"> | null,
  permission: Permission,
): boolean {
  if (user == null) return false
  return permissionsByRole[user.role].includes(permission)
}
```

> 💡 We use `Pick<User, "role">` instead of the full `User` type. This makes the function flexible since you don't need a full user object to call the function.

## Step 2: Handle Complex Permission Logic

Some permissions aren't purely role-based. For example, "can read project" depends on:

- The user's role (admins can read all)
- The user's department
- The project's department

This is where pure RBAC starts to show cracks. We need helper functions that mix role checks with attribute checks:

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

> ⚠️ Notice how we're already mixing RBAC (`can(user, "permission")`) with attribute checks (`project.department === user.department`). This hybrid approach works, but it's a sign that pure RBAC may not be enough for our needs.

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
