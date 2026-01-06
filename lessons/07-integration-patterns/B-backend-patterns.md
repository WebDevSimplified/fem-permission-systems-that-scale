---
description: "Compare middleware-based vs service-layer permission checking approaches."
---

# Backend Patterns

Two main approaches for backend permission checks: middleware and service layer.

## Pattern 1: Middleware (Route-Level)

```typescript
// Check permission before handler runs
router.delete(
  "/:id",
  requireAuth,
  requirePermission("delete", "Document"),
  async (req, res) => {
    await deleteDocument(req.params.id)
    res.status(204).send()
  }
)
```

**Pros:** Clear, declarative, easy to audit routes
**Cons:** Can't access resource data (haven't fetched it yet)

## Pattern 2: Service Layer (Operation-Level)

```typescript
// Check permission inside the handler
router.delete("/:id", requireAuth, async (req, res) => {
  const document = await getDocument(req.params.id)
  const permissions = createPermissions(req.user!)

  permissions.authorize("delete", document) // Throws if denied

  await deleteDocument(document.id)
  res.status(204).send()
})
```

**Pros:** Can check against actual resource, more flexible
**Cons:** Permission logic mixed with business logic

## Pattern 3: Hybrid Approach (Recommended)

```typescript
// Middleware for role check
router.delete(
  "/:id",
  requireAuth,
  requireRole("editor"), // Basic role check
  async (req, res) => {
    const document = await getDocument(req.params.id)
    const permissions = createPermissions(req.user!)

    // Fine-grained check with actual resource
    permissions.authorize("delete", document)

    await deleteDocument(document.id)
    res.status(204).send()
  }
)
```

This catches obviously unauthorized requests early while still supporting resource-level checks.

## Centralized Permission Service

```typescript
// services/permissionService.ts
export class PermissionService {
  constructor(private user: User) {}

  forDocuments() {
    return createDocumentPermissions(this.user)
  }

  forUsers() {
    return createUserPermissions(this.user)
  }
}

// Usage in routes
const permissionService = new PermissionService(req.user!)
permissionService.forDocuments().authorize("update", document)
```
