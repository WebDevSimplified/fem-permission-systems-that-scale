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

## Step 1: Create the ABAC Module

We'll create a new permissions file that defines policies using conditions:

```typescript
// src/permissions/abac.ts

import { Document, Project, User } from "@/drizzle/schema"

export function getUserPermissions(
  user: Pick<User, "department" | "id" | "role"> | null,
) {
  const builder = new PermissionBuilder()
  if (user == null) return builder.build()

  const role = user.role
  switch (role) {
    case "admin":
      addAdminPermissions(builder)
      break
    case "author":
      addAuthorPermissions(builder, user)
      break
    case "editor":
      addEditorPermissions(builder, user)
      break
    case "viewer":
      addViewerPermissions(builder, user)
      break
    default:
      throw new Error(`Unknown role: ${role satisfies never}`)
  }

  return builder.build()
}
```

## Step 2: Define Role-Specific Policies

Each role gets its own policy function with appropriate conditions:

```typescript
function addAdminPermissions(builder: PermissionBuilder) {
  builder
    .allow("project", "read")
    .allow("project", "create")
    .allow("project", "update")
    .allow("project", "delete")
    .allow("document", "read")
    .allow("document", "create")
    .allow("document", "update")
    .allow("document", "delete")
}

function addEditorPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department">,
) {
  builder
    // Can read projects in their department or global projects
    .allow("project", "read", { department: user.department })
    .allow("project", "read", { department: null })
    // Can read all documents, edit unlocked ones
    .allow("document", "read")
    .allow("document", "update", { isLocked: false })
}

function addAuthorPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department" | "id">,
) {
  builder
    .allow("project", "read", { department: user.department })
    .allow("project", "read", { department: null })
    // Can only read published/archived, or their own drafts
    .allow("document", "read", { status: "published" })
    .allow("document", "read", { status: "archived" })
    .allow("document", "read", { status: "draft", creatorId: user.id })
    // Can create documents
    .allow("document", "create")
    // Can only edit their own unlocked drafts
    .allow("document", "update", {
      creatorId: user.id,
      isLocked: false,
      status: "draft",
    })
}

function addViewerPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department">,
) {
  builder
    .allow("project", "read", { department: user.department })
    .allow("project", "read", { department: null })
    .allow("document", "read", { status: "published" })
    .allow("document", "read", { status: "archived" })
}
```

Notice how the conditions are **declarative**—we're describing **what** is allowed, not **how** to check it.

## Step 3: The Permission Builder

The builder pattern makes policies easy to compose:

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
    return this
  }

  build() {
    const permissions = this.#permissions

    return {
      can<Res extends keyof Resources>(
        resource: Res,
        action: Resources[Res]["action"],
        data?: Resources[Res]["condition"],
      ) {
        return permissions[resource].some((perm) => {
          if (perm.action !== action) return false

          // No condition means always allowed for this action
          if (perm.condition == null) return true

          // No data provided means check action permission only
          if (data == null) return true

          // Check all conditions match the resource
          return Object.entries(perm.condition).every(
            ([key, value]) => data[key as keyof typeof data] === value,
          )
        })
      },
    }
  }
}
```

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
