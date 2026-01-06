---
description: "Implement field-level write permissions to control which fields users can modify."
---

# Field-Level Write Permissions

Editors should be able to edit `title` and `content`, but not `status` or `authorId`.

## Field Writability Matrix

| Field               | Editor (own doc) | Admin |
| ------------------- | ---------------- | ----- |
| `title`, `content`  | ✅               | ✅    |
| `status`            | ❌               | ✅    |
| `authorId`, `orgId` | ❌               | ✅    |
| `internalNotes`     | ❌               | ✅    |

## Live Code: Write Field Permissions

```typescript
// server/src/permissions/fields.ts

const WRITABLE_FIELDS: Record<Role, string[]> = {
  viewer: [],
  editor: ["title", "content"],
  admin: ["title", "content", "status", "authorId", "orgId", "internalNotes"],
}

export function getWritableFields(role: Role): string[] {
  return WRITABLE_FIELDS[role]
}

export function filterWriteFields<T extends Record<string, any>>(
  data: T,
  role: Role
): Partial<T> {
  const allowedFields = getWritableFields(role)
  const filtered: Partial<T> = {}

  for (const field of allowedFields) {
    if (field in data) {
      filtered[field as keyof T] = data[field]
    }
  }

  return filtered
}
```

## Live Code: Secure Update Route

```typescript
router.put("/:id", requireAuth, async (req, res) => {
  const document = await db.query.documents.findFirst({
    where: and(
      eq(documentsTable.id, parseInt(req.params.id)),
      eq(documentsTable.orgId, req.user!.orgId)
    ),
  })

  if (!document) {
    return res.status(404).json({ error: "Document not found" })
  }

  if (!canUpdateDocument(req.user!, document)) {
    return res.status(403).json({ error: "Forbidden" })
  }

  // Only allow fields this user can write
  const allowedUpdates = filterWriteFields(req.body, req.user!.role)

  if (Object.keys(allowedUpdates).length === 0) {
    return res.status(400).json({ error: "No valid fields to update" })
  }

  const updated = await db
    .update(documentsTable)
    .set(allowedUpdates)
    .where(eq(documentsTable.id, parseInt(req.params.id)))
    .returning()

  // Filter response fields too
  res.json(filterDocumentFields(updated[0], req.user!.role))
})
```

## Testing

1. As editor, try to update `status` - it's silently ignored
2. As admin, update `status` - it works!
3. Check that editors can still update `title` and `content`
