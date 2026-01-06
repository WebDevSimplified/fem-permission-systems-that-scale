---
description: "Handle complex nested relationships like organization membership implying document access."
---

# Nested Relationships

Real ReBAC handles chains: `User → Org → Document`. Let's implement org-level access.

## The Rule

"Organization members can view all org documents, editors can edit."

```text
User --[member/editor/admin]--> Org --[owns]--> Document
```

## Live Code: Permission Resolution

```typescript
// server/src/permissions/rebac.ts

type Permission = "viewer" | "editor" | "owner" | null

export async function resolveDocumentPermission(
  user: User,
  document: Document
): Promise<Permission> {
  // 1. Direct ownership
  if (document.authorId === user.id) {
    return "owner"
  }

  // 2. Direct share
  const share = await db.query.documentShares.findFirst({
    where: and(
      eq(documentSharesTable.documentId, document.id),
      eq(documentSharesTable.userId, user.id)
    ),
  })
  if (share) return share.permission as Permission

  // 3. Organization membership (same org = viewer)
  if (document.orgId === user.orgId) {
    // Org admins get editor access
    if (user.role === "admin") return "editor"
    // Regular members get viewer
    return "viewer"
  }

  return null
}

export async function canPerformAction(
  user: User,
  document: Document,
  action: "read" | "update" | "delete"
): Promise<boolean> {
  const permission = await resolveDocumentPermission(user, document)

  if (!permission) return false

  switch (action) {
    case "read":
      return true // All permissions allow read
    case "update":
      return permission === "owner" || permission === "editor"
    case "delete":
      return permission === "owner" || user.role === "admin"
  }
}
```

## Permission Hierarchy

```text
owner  → can read, update, delete, share
editor → can read, update
viewer → can read
null   → no access
```

This simple ReBAC covers most sharing use cases without a full graph database.
