---
description: "Demo of ID enumeration attack and how to prevent users from accessing resources by guessing IDs."
---

# 🔴 Mistake: ID Enumeration

Even with ownership checks, there's still a vulnerability: users can discover and access resources from OTHER organizations.

## The Attack

1. Log in as `editor@acme.com` (Acme Corp, orgId: 1)
2. Your documents have IDs like 1, 2, 3
3. Try accessing `/api/documents/10` - a Globex document!
4. Even if you can't edit it, you can READ it

## Why This Happens

Our ownership check only runs on UPDATE. The READ route has no org filtering:

```typescript
// ❌ BAD: Returns ANY document by ID
router.get("/:id", async (req, res) => {
  const doc = await db.query.documents.findFirst({
    where: eq(documentsTable.id, parseInt(req.params.id)),
  })
  res.json(doc) // Leaks cross-org data!
})
```

## The Fix: Organization Scoping

```typescript
// ✅ GOOD: Only returns documents in user's org
router.get("/:id", requireAuth, async (req, res) => {
  const doc = await db.query.documents.findFirst({
    where: and(
      eq(documentsTable.id, parseInt(req.params.id)),
      eq(documentsTable.orgId, req.user!.orgId)
    ),
  })

  if (!doc) {
    return res.status(404).json({ error: "Document not found" })
  }

  res.json(doc)
})
```

Now even if an attacker guesses valid IDs, they get 404 for other orgs' documents.

## Key Principle

> Always scope queries to the user's authorization context.

Don't just check "can they do the action?" Also ensure "is this resource even visible to them?"
