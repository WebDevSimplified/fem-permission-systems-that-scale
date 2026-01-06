---
description: "Learn how to combine multiple conditions into complex policies that handle real-world business rules."
---

# Policy Composition

Real applications have complex rules. Let's compose multiple conditions.

## Example Business Rules

1. Admins can do anything
2. Editors can update their own documents
3. BUT published documents can only be edited by admins
4. AND archived documents cannot be edited by anyone

## Live Code: Composable Policies

```typescript
// server/src/permissions/policies.ts

type PolicyResult = "allow" | "deny" | "skip"

interface PolicyContext {
  user: User
  document: Document
  action: Action
}

type Policy = (ctx: PolicyContext) => PolicyResult

// Individual policies
const adminAllowAll: Policy = ({ user }) =>
  user.role === "admin" ? "allow" : "skip"

const ownerCanUpdate: Policy = ({ user, document, action }) => {
  if (action !== "update") return "skip"
  if (user.role !== "editor") return "skip"
  return document.authorId === user.id ? "allow" : "deny"
}

const publishedReadOnly: Policy = ({ user, document, action }) => {
  if (action !== "update") return "skip"
  if (document.status !== "published") return "skip"
  if (user.role === "admin") return "skip"
  return "deny"
}

const archivedNoEdit: Policy = ({ document, action }) => {
  if (action !== "update" && action !== "delete") return "skip"
  return document.status === "archived" ? "deny" : "skip"
}

// Combine policies
const policies: Policy[] = [
  archivedNoEdit, // Deny first
  publishedReadOnly, // Then restrictions
  adminAllowAll, // Then role allows
  ownerCanUpdate, // Then ownership
]

export function evaluatePolicy(ctx: PolicyContext): boolean {
  for (const policy of policies) {
    const result = policy(ctx)
    if (result === "deny") return false
    if (result === "allow") return true
  }
  return false // Default deny
}
```

## Using the Policy

```typescript
router.put("/:id", requireAuth, async (req, res) => {
  const document = await getDocument(req.params.id, req.user!.orgId)

  const allowed = evaluatePolicy({
    user: req.user!,
    document,
    action: "update",
  })

  if (!allowed) {
    return res.status(403).json({ error: "Forbidden" })
  }

  // Proceed with update...
})
```

## Policy Evaluation Order

```text
1. archivedNoEdit    → DENY if archived (stops here)
2. publishedReadOnly → DENY if published & not admin
3. adminAllowAll     → ALLOW if admin
4. ownerCanUpdate    → ALLOW if owner
5. (default)         → DENY
```

This pattern scales to complex requirements while keeping each rule simple.
