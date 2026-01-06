---
description: "Add organization-level isolation so users can only see and interact with their organization's data."
---

# Organization Scoping

Let's properly isolate organizations using Drizzle's query builder.

## Live Code: Scoped Queries

Create a helper for org-scoped queries:

```typescript
// server/src/permissions/scope.ts
import { eq, and, SQL } from "drizzle-orm"
import { documentsTable } from "../db/schema"

export function scopeToOrg(orgId: number) {
  return eq(documentsTable.orgId, orgId)
}

export function scopeToAuthor(authorId: number) {
  return eq(documentsTable.authorId, authorId)
}
```

## Live Code: Update All Document Routes

```typescript
// GET /api/documents - List only user's org documents
router.get("/", requireAuth, async (req, res) => {
  const documents = await db
    .select()
    .from(documentsTable)
    .where(eq(documentsTable.orgId, req.user!.orgId))

  res.json(documents)
})

// GET /api/documents/:id - Single document, org-scoped
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

  res.json(document)
})
```

## Testing Organization Isolation

1. Log in as `editor@acme.com`
2. List documents - only see Acme documents
3. Try `/api/documents/[globex-doc-id]` - 404!
4. Log in as `editor@globex.com`
5. List documents - only see Globex documents

Organizations are now properly isolated.
