---
description: "A subtle race condition in permission checking and how to prevent it."
---

# 🔴 Mistake: TOCTOU Race Condition

Time-of-Check to Time-of-Use (TOCTOU) is a subtle but dangerous vulnerability.

## The Problem

```typescript
router.put("/:id", requireAuth, async (req, res) => {
  const document = await getDocument(req.params.id)

  // CHECK: Is user allowed to update?
  if (!canUpdate(req.user!, document)) {
    return res.status(403).json({ error: "Forbidden" })
  }

  // ... some async operations ...
  await someOtherOperation()

  // USE: Perform the update
  // ⚠️ Document may have changed since we checked!
  await db
    .update(documentsTable)
    .set(req.body)
    .where(eq(documentsTable.id, document.id))
})
```

## The Attack

1. Editor requests to update their document (id: 5)
2. Server checks: document.authorId === user.id ✓
3. Admin changes document 5's authorId to someone else
4. Server proceeds with update (permission was checked against OLD data)

## Solutions

### Solution 1: Transaction with Re-check

```typescript
await db.transaction(async (tx) => {
  // Get document WITH lock
  const document = await tx.query.documents.findFirst({
    where: eq(documentsTable.id, parseInt(req.params.id)),
  })

  // Re-check permission inside transaction
  if (!canUpdate(req.user!, document)) {
    throw new Error("Forbidden")
  }

  // Update in same transaction
  await tx
    .update(documentsTable)
    .set(req.body)
    .where(eq(documentsTable.id, document.id))
})
```

### Solution 2: Conditional Update

```typescript
// Include permission conditions in the WHERE clause
const result = await db
  .update(documentsTable)
  .set(req.body)
  .where(
    and(
      eq(documentsTable.id, parseInt(req.params.id)),
      eq(documentsTable.authorId, req.user!.id) // Permission condition
    )
  )

if (result.rowsAffected === 0) {
  return res.status(403).json({ error: "Forbidden or not found" })
}
```

This is atomic - check and update happen together!

## Key Takeaway

> When possible, encode permission checks into database queries rather than checking in application code before the operation.
