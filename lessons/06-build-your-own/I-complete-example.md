---
description: "Put it all together and see our custom permission library in action."
---

# Complete Example

Let's see our entire permission library working together.

## Full Permission Definition

```typescript
// shared/permissions/documentPermissions.ts

import { createPermissions } from "@our-app/permissions"
import type { User } from "../types"

export function createDocumentPermissions(user: User) {
  return createPermissions<User, "Document" | "Comment">(
    (allow, deny, { user: u }) => {
      // === READ PERMISSIONS ===

      // Everyone can read documents in their org
      allow("read", "Document", { orgId: u.orgId })

      // Field permissions for reading
      allow("read", "Document", [
        "id",
        "title",
        "content",
        "status",
        "createdAt",
      ])

      if (u.role === "editor" || u.role === "admin") {
        allow("read", "Document", ["authorId"])
      }

      if (u.role === "admin") {
        allow("read", "Document", ["orgId", "internalNotes"])
      }

      // === WRITE PERMISSIONS ===

      if (u.role === "editor") {
        allow("create", "Document")
        allow("update", "Document", { authorId: u.id, orgId: u.orgId })
        // Editors can only write these fields
        allow("write", "Document", ["title", "content"])
      }

      if (u.role === "admin") {
        allow("manage", "Document", { orgId: u.orgId })
        allow("write", "Document", [
          "title",
          "content",
          "status",
          "internalNotes",
        ])
      }

      // === DENY RULES ===

      // Archived documents cannot be modified
      deny("update", "Document", { status: "archived" })
      deny("delete", "Document", { status: "archived" })
    },
    user
  )
}
```

## Backend Usage

```typescript
// server/src/routes/documents.ts

router.get("/", requireAuth, async (req, res) => {
  const permissions = createDocumentPermissions(req.user!)

  const docs = await db
    .select()
    .from(documentsTable)
    .where(permissions.filter("read", "Document", documentsTable))

  // Filter fields on each document
  const filtered = docs.map((doc) => permissions.filterFields(doc))

  res.json(filtered)
})

router.put("/:id", requireAuth, async (req, res) => {
  const permissions = createDocumentPermissions(req.user!)
  const document = await getDocument(req.params.id)

  // Check action permission
  permissions.authorize("update", document)

  // Filter to only allowed write fields
  const allowedFields = permissions.allowedFields("write", "Document")
  const updates = pick(req.body, allowedFields)

  await db
    .update(documentsTable)
    .set(updates)
    .where(eq(documentsTable.id, document.id))

  res.json({ success: true })
})
```

## Frontend Usage

```tsx
function DocumentList() {
  const { documents } = useDocuments()

  return (
    <div>
      <Can action="create" subject="Document">
        <Link to="/documents/new">Create Document</Link>
      </Can>

      {documents.map((doc) => (
        <DocumentCard key={doc.id} document={doc} />
      ))}
    </div>
  )
}
```

We now have a complete, type-safe permission system! 🎉
