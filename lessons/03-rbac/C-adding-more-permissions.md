---
title: "Adding More Permissions"
---

Our RBAC system works great—until the business asks for "just a few more rules." Let's add some realistic permission requirements and see how RBAC handles them.

## New Permission Requirements

Our system is currently very bare-bones and a more realistic set of permissions may include the following:

### 1. Locked Documents

Documents can be **locked** to prevent editing. Only admins can edit locked documents.

### 2. Draft Visibility

**Draft documents** should only be visible to:

- The document creator (authors can see their own drafts)
- Editors and admins (they can see all drafts)
- Viewers should **not** see drafts

### 3. Author Edit Restrictions

Authors should only be able to edit:

- Documents **they created**
- That are **not locked**
- That are still in **draft status**

Once a document is published, only editors and admins can modify it.

### Full Permission Matrix

Here's the complete permission matrix we need to implement:

| Action                           | Admin | Editor | Author | Viewer |
| -------------------------------- | ----- | ------ | ------ | ------ |
| **View** published/archived docs | ✅    | ✅     | ✅     | ✅     |
| **View** draft docs (any)        | ✅    | ✅     | ❌     | ❌     |
| **View** draft docs (own)        | ✅    | ✅     | ✅     | ❌     |
| **Edit** unlocked docs           | ✅    | ✅     | ❌     | ❌     |
| **Edit** own unlocked draft docs | ✅    | ✅     | ✅     | ❌     |
| **Edit** locked docs             | ✅    | ❌     | ❌     | ❌     |
| **Delete** docs                  | ✅    | ❌     | ❌     | ❌     |
| **Create** docs                  | ✅    | ❌     | ✅     | ❌     |

## The Problem Emerges

Look at the "Author Edit" requirement again:

> Authors can edit documents **they created**, that are **not locked**, in **draft status**

This permission depends on:

- The **user's role** (must be author)
- The **user's ID** (must be the creator)
- The **document's isLocked** attribute
- The **document's status** attribute

Pure RBAC struggles to express this. We need to check **attributes** of both the user and the resource.

## Attempting RBAC Solutions

To solve this in RBAC, we need to create very specific permissions:

```typescript
type Permission =
  // ...existing permissions...
  | "document:update:unlocked"
  | "document:update:own-unlocked-draft"
  | "document:read:all"
  | "document:read:own"
  | "document:read:non-draft"
```

Then assign them to roles:

```typescript
const permissionsByRole = {
  admin: [
    "document:update:unlocked", // Can edit any unlocked doc
    "document:read:all",
    // ...
  ],
  author: [
    "document:update:own-unlocked-draft", // Very specific!
    "document:read:own",
    "document:read:non-draft",
    // ...
  ],
  // ...
}
```

But now we need helper functions to use these effectively:

```typescript
export function canUpdateDocument(
  user: Pick<User, "role" | "id"> | null,
  document: Pick<Document, "creatorId" | "status" | "isLocked">,
): boolean {
  if (user == null) return false

  return (
    // Admins/editors: can edit any unlocked document
    (can(user, "document:update:unlocked") && !document.isLocked) ||
    // Authors: only their own, unlocked, draft documents
    (can(user, "document:update:own-unlocked-draft") &&
      document.creatorId === user.id &&
      !document.isLocked &&
      document.status === "draft")
  )
}
```

## What Went Wrong?

Notice the problems:

### 1. Permissions Exploded In Complexity

We now have 3 permissions to read a document, 2 for updating, and 3 for reading a project. This problem also grows exponentially as more attributes and conditions are added since if we needed to check just 2 more attributes on a document we would need to create permissions for every permutation of those attributes.

### 2. Logic Scattered Again

Our simple `can` function is no longer able to handle complex permission checks on its own which leads to many helper functions that are complex and easy to misuse.

### 3. Database Queries Duplicate Permissions

With the current RBAC implementation, we need to essentially rewrite each permission check into its corresponding database query when querying multiple items (such as getting all the projects that user has access to). This leads to duplicated logic between the permission system code and the database layer, making it harder to maintain and increasing the risk of inconsistencies.

## Branch Checkpoint

After adding these permissions, your code should match:

```text
Branch: 4-rbac-limits
```

Run the following to sync up:

```bash
git checkout 4-rbac-limits
```
