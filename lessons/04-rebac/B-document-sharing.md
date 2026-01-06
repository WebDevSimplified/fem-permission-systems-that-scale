---
description: "Implement document sharing with a relationship table and permission checks."
---

# Document Sharing

Let's add a "Share" feature where users can grant others access to their documents.

## Live Code: Share Table Schema

```typescript
// server/src/db/schema.ts

export const documentSharesTable = sqliteTable("document_shares", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  documentId: integer("document_id")
    .notNull()
    .references(() => documentsTable.id, { onDelete: "cascade" }),
  userId: integer("user_id")
    .notNull()
    .references(() => usersTable.id, { onDelete: "cascade" }),
  permission: text("permission", { enum: ["viewer", "editor"] }).notNull(),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .default(sql`(unixepoch())`),
})
```

## Live Code: Share Permissions

```typescript
// server/src/permissions/shares.ts

export async function getDocumentPermission(
  userId: number,
  documentId: number
): Promise<"owner" | "editor" | "viewer" | null> {
  // Check if owner
  const doc = await db.query.documents.findFirst({
    where: eq(documentsTable.id, documentId),
  })

  if (doc?.authorId === userId) return "owner"

  // Check shares
  const share = await db.query.documentShares.findFirst({
    where: and(
      eq(documentSharesTable.documentId, documentId),
      eq(documentSharesTable.userId, userId)
    ),
  })

  return share?.permission ?? null
}

export async function canAccessDocument(
  userId: number,
  documentId: number
): Promise<boolean> {
  const permission = await getDocumentPermission(userId, documentId)
  return permission !== null
}
```

## Live Code: Share API

```typescript
// POST /api/documents/:id/share
router.post("/:id/share", requireAuth, async (req, res) => {
  const { userId, permission } = req.body
  const documentId = parseInt(req.params.id)

  // Only owner can share
  const doc = await getDocument(documentId)
  if (doc.authorId !== req.user!.id && req.user!.role !== "admin") {
    return res.status(403).json({ error: "Only owners can share" })
  }

  await db.insert(documentSharesTable).values({
    documentId,
    userId,
    permission,
  })

  res.json({ success: true })
})
```

Now users can share documents with specific permissions!
