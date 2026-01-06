---
description: "Understand when to use code-based permissions vs a UI-based permission editor."
---

# Why a Permission Editor?

So far, our permissions are defined in code. But what if non-developers need to manage them?

## Code-Based Permissions

```typescript
// permissions/documentPermissions.ts
if (user.role === "editor") {
  allow("update", "Document", { authorId: user.id })
}
```

**Pros:**

- ✅ Type-safe, IDE support
- ✅ Version controlled (Git)
- ✅ Can be unit tested
- ✅ No runtime errors from bad config
- ✅ Reviewed in PRs

**Cons:**

- ❌ Requires developer to change
- ❌ Requires deployment
- ❌ Can't respond quickly to business needs

## UI-Based Permissions

**Pros:**

- ✅ Non-developers can manage
- ✅ Instant changes, no deployment
- ✅ Built-in audit trail
- ✅ Self-service for business teams

**Cons:**

- ❌ Can create "permission spaghetti"
- ❌ Harder to test systematically
- ❌ Risk of misconfigurations
- ❌ Need to build & maintain the UI

## When to Use Each

| Scenario                            | Recommendation                               |
| ----------------------------------- | -------------------------------------------- |
| Small team, simple permissions      | Code-based                                   |
| Enterprise, compliance requirements | UI + audit trail                             |
| Frequently changing rules           | UI-based                                     |
| Security-critical permissions       | Code-based (reviewed)                        |
| Multi-tenant SaaS                   | Hybrid: base in code, tenant overrides in DB |

## Our Approach

We'll build a simple visual editor that:

1. Stores permission rules in the database
2. Provides dropdowns for role, action, subject
3. Checkboxes for field permissions
4. Simple condition builder (field = value)

Let's start with the database schema.
