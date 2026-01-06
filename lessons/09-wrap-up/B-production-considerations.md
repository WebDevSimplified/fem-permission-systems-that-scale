---
description: "Performance optimization, audit logging, and other considerations for production permission systems."
---

# Production Considerations

Before deploying your permission system, consider these production concerns.

## Performance

### Cache Abilities

Don't rebuild permissions on every request:

```typescript
// Per-request cache (simple)
const abilityCache = new WeakMap<User, Permissions>()

function getAbility(user: User) {
  if (!abilityCache.has(user)) {
    abilityCache.set(user, createDocumentPermissions(user))
  }
  return abilityCache.get(user)!
}
```

### Optimize Database Filters

Use indexes on commonly filtered columns:

```sql
CREATE INDEX idx_documents_org ON documents(org_id);
CREATE INDEX idx_documents_author ON documents(author_id);
CREATE INDEX idx_documents_status ON documents(status);
```

## Audit Logging

Log permission checks for security audits:

```typescript
function logPermissionCheck(
  user: User,
  action: string,
  subject: string,
  result: boolean,
  reason?: string
) {
  logger.info("Permission check", {
    userId: user.id,
    action,
    subject,
    result: result ? "ALLOW" : "DENY",
    reason,
    timestamp: new Date().toISOString(),
  })
}
```

## Testing

Write explicit tests for your permission rules:

```typescript
describe("Document permissions", () => {
  it("editors can update their own documents", () => {
    const editor = { id: 1, role: "editor", orgId: 1 }
    const ownDoc = { __typename: "Document", authorId: 1, orgId: 1 }
    const otherDoc = { __typename: "Document", authorId: 2, orgId: 1 }

    const permissions = createDocumentPermissions(editor)

    expect(permissions.can("update", ownDoc)).toBe(true)
    expect(permissions.can("update", otherDoc)).toBe(false)
  })

  it("viewers cannot create documents", () => {
    const viewer = { id: 1, role: "viewer", orgId: 1 }
    const permissions = createDocumentPermissions(viewer)

    expect(permissions.can("create", "Document")).toBe(false)
  })
})
```

## Error Messages

Be careful with error message detail:

```typescript
// ❌ Too detailed (information leakage)
res.status(403).json({
  error: "You cannot edit document 123 because you are not the author",
})

// ✅ Safe
res.status(403).json({ error: "Forbidden" })

// Log the detail server-side for debugging
logger.warn("Permission denied", { userId, documentId, reason: "not author" })
```
