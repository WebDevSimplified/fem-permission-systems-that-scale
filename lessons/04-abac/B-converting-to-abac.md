---
title: "Converting to ABAC"
---

Let's convert our RBAC system to ABAC. This first pass will have **identical functionality**—we're just restructuring the code to use an attribute-based model instead of a role-based one.

## The Goal

Replace this hybrid RBAC code:

```typescript
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

With a unified ABAC API:

```typescript
// Single ABAC check handles everything
if (!permissions.can("document", "update", document)) {
  throw new AuthorizationError()
}
```

## How ABAC Works

In ABAC, we evaluate **attributes** from multiple sources to make access decisions:

![ABAC Flow](/fem-permission-systems-that-scale/images/04-abac/abac-flow.svg)

Policies evaluate attributes from both subject and resource to make decisions. The key difference from RBAC: instead of looking up permissions by role alone, we evaluate **conditions** against the actual data at runtime.

## Step 1: Create the ABAC Module

Here's the core idea in minimal code:

```typescript
// 1. Build permissions with conditions
const builder = new PermissionBuilder()
builder.allow("document", "update", { creatorId: user.id, isLocked: false })
const permissions = builder.build()

// 2. Check against actual resource data
permissions.can("document", "update", document) // true if all conditions match
```

The `allow()` call says "this user can update documents **where** they're the creator AND it's not locked." The `can()` call evaluates those conditions against the actual document.

### The Entry Point

We'll create `src/permissions/abac.ts`. The entry point routes to role-specific policy builders:

```typescript
// src/permissions/abac.ts

import { Document, Project, User } from "@/drizzle/schema"

export function getUserPermissions(
  user: Pick<User, "department" | "id" | "role"> | null,
) {
  const builder = new PermissionBuilder()
  if (user == null) return builder.build()

  // Add role-specific permissions...

  return builder.build()
}
```

## Step 2: Define Role-Specific Policies

Next we need to define the policies for each role.

### Editor Permissions Example

```typescript
builder
  // Can read projects in their department or global projects
  .allow("project", "read", { department: user.department })
  .allow("project", "read", { department: null })
  // Can read all documents, edit unlocked ones
  .allow("document", "read")
  .allow("document", "update", { isLocked: false })
```

Look at that last `.allow()` call: "editors can update documents that are unlocked." In RBAC, this would require a helper function with manual attribute checks. In ABAC, it's declarative.

## Step 3: The Permission Builder

The builder pattern makes policies easy to compose. Let's build it piece by piece.

### Type Definitions

First, we define what resources exist and what attributes they have:

```typescript
type Resources = {
  project: {
    action: "create" | "read" | "update" | "delete"
    condition: Pick<Project, "department">
  }
  document: {
    action: "create" | "read" | "update" | "delete"
    condition: Pick<Document, "projectId" | "creatorId" | "status" | "isLocked">
  }
}
```

This gives us type safety: TypeScript knows that `project` conditions can only include `department`, while `document` conditions can include `status`, `isLocked`, etc.

### The `allow()` Method

The `allow()` method accumulates permission rules:

```typescript
class PermissionBuilder {
  #permissions: PermissionStore = {
    project: [],
    document: [],
  }

  allow<Res extends keyof Resources>(
    resource: Res,
    action: Resources[Res]["action"],
    condition?: Partial<Resources[Res]["condition"]>,
  ) {
    this.#permissions[resource].push({ action, condition })
    return this // Enable chaining: .allow(...).allow(...)
  }
}
```

### The `can()` Method

The `can()` method is where the magic happens—it evaluates conditions against actual data:

```typescript
build() {
  const permissions = this.#permissions

  return {
    can<Res extends keyof Resources>(
      resource: Res,
      action: Resources[Res]["action"],
      data?: Resources[Res]["condition"],
    ) {
      return permissions[resource].some((perm) => {
        // Must match the requested action
        if (perm.action !== action) return false

        // No condition = always allowed for this action
        if (perm.condition == null) return true

        // No data provided = just checking if action is possible
        if (data == null) return true

        // Check all conditions match the resource
        return Object.entries(perm.condition).every(
          ([key, value]) => data[key as keyof typeof data] === value,
        )
      })
    },
  }
}
```

The algorithm:

1. Find any permission matching the action
2. If that permission has no conditions, allow immediately
3. Otherwise, verify every condition matches the resource's attributes

## Step 4: Update the Codebase

Now we replace all permission checks with the new API.

### Before (Page Component)

```typescript
import { can } from "@/permissions/rbac"
import { canReadProject } from "@/permissions/projects"
import { canUpdateDocument } from "@/permissions/documents"

const user = await getCurrentUser()
if (!canReadProject(user, project)) {
  return redirect("/")
}
if (!canUpdateDocument(user, document)) {
  return redirect(`/projects/${projectId}`)
}
```

### After (Page Component)

```typescript
import { getUserPermissions } from "@/permissions/abac"

const user = await getCurrentUser()
const permissions = getUserPermissions(user)

if (!permissions.can("project", "read", project)) {
  return redirect("/")
}
if (!permissions.can("document", "update", document)) {
  return redirect(`/projects/${projectId}`)
}
```

### Before (Mutation)

```typescript
import { can } from "@/permissions/rbac"
import { canUpdateDocument } from "@/permissions/documents"

const user = await getCurrentUser()
const document = await getDocumentById(documentId)
if (!canUpdateDocument(user, document)) {
  throw new AuthorizationError()
}
```

### After (Mutation)

```typescript
import { getUserPermissions } from "@/permissions/abac"

const user = await getCurrentUser()
const permissions = getUserPermissions(user)
const document = await getDocumentById(documentId)

if (!permissions.can("document", "update", document)) {
  throw new AuthorizationError()
}
```

## What We Gain

| Benefit                  | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **Unified API**          | One `can()` function for all checks                          |
| **Declarative policies** | Conditions describe what's allowed                           |
| **No helper functions**  | No more `canUpdateDocument`, `canReadProject`, etc.          |
| **Type safety**          | TypeScript enforces valid resources, actions, and conditions |
| **Composable**           | Easy to add new conditions without new permission strings    |

## What We Delete

With ABAC in place, we can remove:

- `src/permissions/rbac.ts` - The old RBAC module
- `src/permissions/documents.ts` - The document helper functions
- `src/permissions/projects.ts` - The project helper functions

Everything is now in `src/permissions/abac.ts`.

## Branch Checkpoint

After completing this conversion, your code should match:

```text
Branch: 5-abac-basic
```

Run the following to sync up:

```bash
git checkout 5-abac-basic
```

## What's Next

This basic ABAC system has the same functionality as our RBAC + helpers approach, just with cleaner architecture. In the next lesson, we'll add **advanced features** that ABAC makes possible—like field-level permissions and automatic query filtering.
