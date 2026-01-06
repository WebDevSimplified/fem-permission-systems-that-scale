---
description: "Implement ownership checks so editors can only edit documents they created."
---

# Ownership-Based Permissions

Our first ABAC rule: editors can only edit their OWN documents.

## Live Code: Permission Check Function

Update `server/src/permissions/documents.ts`:

```typescript
import { type Role } from "./roles"

interface User {
  id: number
  role: Role
  orgId: number
}

interface Document {
  id: number
  authorId: number
  orgId: number
  status: "draft" | "published" | "archived"
}

export function canUpdateDocument(user: User, document: Document): boolean {
  // Admins can update anything
  if (user.role === "admin") {
    return true
  }

  // Editors can only update their own documents
  if (user.role === "editor") {
    return document.authorId === user.id
  }

  // Viewers cannot update
  return false
}

export function canDeleteDocument(user: User, document: Document): boolean {
  // Only admins can delete
  return user.role === "admin"
}
```

## Live Code: Apply to Routes

Update the PUT route in `server/src/routes/documents.ts`:

```typescript
router.put("/:id", requireAuth, async (req, res) => {
  const { id } = req.params

  // First, fetch the document to check ownership
  const document = await db.query.documents.findFirst({
    where: eq(documentsTable.id, parseInt(id)),
  })

  if (!document) {
    return res.status(404).json({ error: "Document not found" })
  }

  // Check permission with ABAC
  if (!canUpdateDocument(req.user!, document)) {
    return res.status(403).json({
      error: "Forbidden",
      message: "You can only edit your own documents",
    })
  }

  // Proceed with update
  const updated = await db
    .update(documentsTable)
    .set({ title: req.body.title, content: req.body.content })
    .where(eq(documentsTable.id, parseInt(id)))
    .returning()

  res.json(updated[0])
})
```

## Testing Ownership

1. Log in as `editor@acme.com`
2. Create a new document (you own it)
3. Edit it - **works!**
4. Find a document by `admin@acme.com`
5. Try to edit it - **403 Forbidden!**

Now editors can only modify their own work.
