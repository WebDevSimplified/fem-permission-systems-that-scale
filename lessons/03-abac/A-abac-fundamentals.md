---
description: "Learn the core concepts of ABAC: attributes, policies, and how to make context-aware permission decisions."
---

# ABAC Fundamentals

Attribute-Based Access Control extends RBAC by considering **attributes** of the user, resource, and environment when making decisions.

## The ABAC Formula

```text
ALLOW if policy(subject_attributes, resource_attributes, action, context) = true
```

Instead of just "does role have permission?", we ask "do ALL the conditions match?"

## Types of Attributes

### Subject Attributes (User)

- `id` - User's unique identifier
- `role` - Their role (admin, editor, viewer)
- `orgId` - Their organization
- `department` - Their department
- `clearanceLevel` - Security clearance

### Resource Attributes (Document)

- `id` - Document identifier
- `authorId` - Who created it
- `orgId` - Which organization owns it
- `status` - draft, published, archived
- `classification` - public, internal, confidential

### Environment/Context Attributes

- `time` - Current time (business hours?)
- `ipAddress` - Where request comes from
- `deviceType` - Mobile or desktop

## Example Policies

```typescript
// Policy 1: Editors can update their own documents
if (user.role === "editor" && document.authorId === user.id) {
  allow("update")
}

// Policy 2: Users can only see documents in their org
if (document.orgId === user.orgId) {
  allow("read")
}

// Policy 3: Published documents are read-only for non-admins
if (document.status === "published" && user.role !== "admin") {
  deny("update")
}
```

## ABAC vs RBAC

| Aspect          | RBAC      | ABAC                |
| --------------- | --------- | ------------------- |
| Decision input  | Role only | Multiple attributes |
| Granularity     | Coarse    | Fine                |
| "Own resources" | ❌        | ✅                  |
| Dynamic rules   | ❌        | ✅                  |
| Complexity      | Low       | Medium              |

Let's implement ownership-based permissions first.
