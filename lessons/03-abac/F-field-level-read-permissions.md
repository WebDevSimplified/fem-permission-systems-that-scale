---
description: "Implement field-level read permissions to hide sensitive data from unauthorized users."
---

# Field-Level Read Permissions

Not all users should see all fields. Let's control what data is returned based on role.

## Field Visibility Matrix

| Field                    | Viewer | Editor | Admin |
| ------------------------ | ------ | ------ | ----- |
| `id`, `title`, `content` | ✅     | ✅     | ✅    |
| `status`, `createdAt`    | ✅     | ✅     | ✅    |
| `authorId`               | ❌     | ✅     | ✅    |
| `orgId`, `internalNotes` | ❌     | ❌     | ✅    |

## Live Code: Field Filtering

```typescript
// server/src/permissions/fields.ts
import { type Role } from "./roles"

const READABLE_FIELDS: Record<Role, string[]> = {
  viewer: ["id", "title", "content", "status", "createdAt"],
  editor: ["id", "title", "content", "status", "createdAt", "authorId"],
  admin: [
    "id",
    "title",
    "content",
    "status",
    "createdAt",
    "authorId",
    "orgId",
    "internalNotes",
  ],
}

export function getReadableFields(role: Role): string[] {
  return READABLE_FIELDS[role]
}

export function filterDocumentFields<T extends Record<string, any>>(
  document: T,
  role: Role
): Partial<T> {
  const allowedFields = getReadableFields(role)
  const filtered: Partial<T> = {}

  for (const field of allowedFields) {
    if (field in document) {
      filtered[field as keyof T] = document[field]
    }
  }

  return filtered
}
```

## Live Code: Apply to Routes

```typescript
router.get("/:id", requireAuth, async (req, res) => {
  const document = await db.query.documents.findFirst({
    where: and(
      eq(documentsTable.id, parseInt(req.params.id)),
      eq(documentsTable.orgId, req.user!.orgId)
    ),
  })

  if (!document) {
    return res.status(404).json({ error: "Document not found" })
  }

  // Filter fields based on role
  const filtered = filterDocumentFields(document, req.user!.role)
  res.json(filtered)
})
```

## Testing

- Viewer sees: `{ id, title, content, status, createdAt }`
- Editor sees: same + `authorId`
- Admin sees: everything including `internalNotes`
