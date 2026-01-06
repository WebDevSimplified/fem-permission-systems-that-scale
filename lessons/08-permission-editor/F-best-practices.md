---
description: "Best practices for permission editors including safety guardrails and audit logging."
---

# Best Practices

Permission editors are powerful. Here's how to use them safely.

## 1. Prevent Admin Lockout

Never let admins remove their own admin access:

```typescript
// Before saving a rule deletion
if (rule.role === "admin" && rule.action === "manage") {
  const remainingAdminRules = await db.query.permissionRules.findFirst({
    where: and(
      eq(permissionRulesTable.role, "admin"),
      eq(permissionRulesTable.action, "manage"),
      ne(permissionRulesTable.id, rule.id)
    ),
  })

  if (!remainingAdminRules) {
    throw new Error("Cannot remove last admin permission")
  }
}
```

## 2. Audit Trail

Log all permission changes:

```typescript
export const permissionAuditTable = sqliteTable("permission_audit", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  ruleId: integer("rule_id"),
  action: text("action"), // 'created', 'updated', 'deleted'
  previousValue: text("previous_value", { mode: "json" }),
  newValue: text("new_value", { mode: "json" }),
  changedBy: integer("changed_by").references(() => usersTable.id),
  changedAt: integer("changed_at", { mode: "timestamp" }).notNull(),
})
```

## 3. Caching

Don't query DB on every request:

```typescript
const permissionCache = new Map<string, { rules: Rule[]; expiry: number }>()

async function getCachedRules(role: string) {
  const cached = permissionCache.get(role)
  if (cached && cached.expiry > Date.now()) {
    return cached.rules
  }

  const rules = await db.query.permissionRules.findMany({
    where: eq(permissionRulesTable.role, role),
  })

  permissionCache.set(role, { rules, expiry: Date.now() + 60000 }) // 1 min cache
  return rules
}
```

## 4. Hybrid Approach

Keep critical permissions in code, allow UI overrides:

```typescript
const permissions = createPermissions((allow, deny, { user }) => {
  // Base permissions (always in code)
  allow("read", "Document", { orgId: user.orgId })

  // Dynamic overrides from database
  for (const rule of dynamicRules) {
    // ... apply dynamic rules
  }

  // Hardcoded safety rules (can't be overridden)
  deny("delete", "User", { id: user.id }) // Can't delete yourself
}, user)
```

This gives flexibility while maintaining guardrails.
