---
description: "See the complete permission flow from login to database query to UI rendering."
---

# Full Flow Demo

Let's trace a complete permission-aware request through our system.

## The Request

User `editor@acme.com` loads the document list.

## Step 1: Authentication

```typescript
// Middleware extracts user from session
req.user = {
  id: 2,
  email: "editor@acme.com",
  role: "editor",
  orgId: 1, // Acme Corp
}
```

## Step 2: Build Permissions

```typescript
// In route handler
const permissions = createDocumentPermissions(req.user)

// This runs the builder:
allow("read", "Document", { orgId: 1 }) // Their org
allow("read", "Document", [
  "id",
  "title",
  "content",
  "status",
  "createdAt",
  "authorId",
])
allow("create", "Document")
allow("update", "Document", { authorId: 2, orgId: 1 }) // Their docs
```

## Step 3: Generate Database Filter

```typescript
const filter = permissions.filter("read", "Document", documentsTable)
// Generates: WHERE orgId = 1
```

## Step 4: Query with Filter

```typescript
const documents = await db.select().from(documentsTable).where(filter)
// Returns only Acme Corp documents
```

## Step 5: Filter Response Fields

```typescript
const response = documents.map((doc) => permissions.filterFields(doc))
// Each doc has: id, title, content, status, createdAt, authorId
// Removed: orgId, internalNotes (editor can't see these)
```

## Step 6: Client Receives Data

```json
[
  { "id": 1, "title": "Q4 Report", "authorId": 1, "status": "published" },
  { "id": 2, "title": "Draft Ideas", "authorId": 2, "status": "draft" }
]
```

## Step 7: React Renders with Permissions

```tsx
// Client builds same permissions from user context
{
  documents.map((doc) => (
    <div key={doc.id}>
      <h3>{doc.title}</h3>

      <Can action="update" subject={doc}>
        {/* Only shows for doc.id=2 (their doc) */}
        <button>Edit</button>
      </Can>
    </div>
  ))
}
```

## Result

- User only sees Acme documents (org filter)
- Sensitive fields are hidden (field filter)
- Edit button only appears on their own documents (UI check)
- Backend would reject edit on doc #1 anyway (defense in depth)

This is defense in depth: multiple layers all enforcing the same rules.
