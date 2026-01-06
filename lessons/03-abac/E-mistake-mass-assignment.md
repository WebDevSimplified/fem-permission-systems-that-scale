---
description: "Demo of mass assignment vulnerability where users can modify fields they shouldn't have access to."
---

# 🔴 Mistake: Mass Assignment

Even with ownership and org scoping, there's another attack: modifying fields you shouldn't be able to change.

## The Attack

```javascript
// Editor sends this update request:
fetch("/api/documents/5", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    title: "My Document",
    content: "Content...",
    authorId: 1, // Trying to claim admin's document!
    status: "published", // Bypassing review process!
  }),
})
```

## The Vulnerable Code

```typescript
// ❌ BAD: Directly using request body
router.put("/:id", async (req, res) => {
  await db
    .update(documentsTable)
    .set(req.body) // Takes ALL fields from request!
    .where(eq(documentsTable.id, req.params.id))
})
```

## The Fix: Whitelist Allowed Fields

```typescript
// ✅ GOOD: Only allow specific fields
router.put("/:id", async (req, res) => {
  const allowedFields = getAllowedWriteFields(req.user!)

  const updates: Partial<Document> = {}
  for (const field of allowedFields) {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field]
    }
  }

  await db
    .update(documentsTable)
    .set(updates)
    .where(eq(documentsTable.id, req.params.id))
})
```

## Key Principle

> Never trust user input. Always whitelist allowed fields.

This applies to ALL models, especially User (prevent role escalation!).
