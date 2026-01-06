---
description: "Create the role and permission type definitions in TypeScript that will be the foundation of our authorization system."
---

# Defining Roles and Permissions

Let's create our first permission code. We'll define the roles and what each role can do.

## Live Code: Create the Permissions File

Create a new file at `server/src/permissions/roles.ts`:

```typescript
// server/src/permissions/roles.ts

// Define all possible actions in our system
export const ACTIONS = [
  "read",
  "create",
  "update",
  "delete",
  "manage", // Special action for admin-only operations
] as const

export type Action = (typeof ACTIONS)[number]

// Define role-to-permission mapping
export const ROLES = {
  viewer: ["read"],
  editor: ["read", "create", "update"],
  admin: ["read", "create", "update", "delete", "manage"],
} as const satisfies Record<string, readonly Action[]>

export type Role = keyof typeof ROLES

// Helper function to check if a role has a permission
export function hasPermission(role: Role, action: Action): boolean {
  const permissions = ROLES[role]
  return permissions.includes(action)
}

// Helper to check if a role is at least as powerful as another
export function isAtLeast(userRole: Role, requiredRole: Role): boolean {
  const hierarchy: Role[] = ["viewer", "editor", "admin"]
  return hierarchy.indexOf(userRole) >= hierarchy.indexOf(requiredRole)
}
```

## Understanding the Types

Let's break down what we just created:

### `ACTIONS`

```typescript
export const ACTIONS = ["read", "create", "update", "delete", "manage"] as const
```

The `as const` assertion makes this a readonly tuple, which allows TypeScript to infer literal types instead of just `string[]`.

### `Action` Type

```typescript
export type Action = (typeof ACTIONS)[number]
// Result: 'read' | 'create' | 'update' | 'delete' | 'manage'
```

This creates a union type of all valid actions. If you try to use an invalid action, TypeScript will catch it at compile time.

### `ROLES` with `satisfies`

```typescript
export const ROLES = {
  viewer: ["read"],
  // ...
} as const satisfies Record<string, readonly Action[]>
```

The `satisfies` keyword ensures our role definitions match the expected shape while preserving the literal types. This gives us:

- Type checking that all permissions are valid actions
- Autocomplete when using role names
- Literal types preserved (not widened to `string[]`)

## Testing the Types

```typescript
import { hasPermission, isAtLeast, type Role, type Action } from "./roles"

// TypeScript knows these are valid
const role: Role = "editor" // ✅
const action: Action = "update" // ✅

// TypeScript catches these errors
const badRole: Role = "superuser" // ❌ Error!
const badAction: Action = "fly" // ❌ Error!

// Runtime checks
hasPermission("editor", "read") // true
hasPermission("editor", "delete") // false
hasPermission("admin", "manage") // true

isAtLeast("admin", "editor") // true
isAtLeast("viewer", "editor") // false
```

## Why This Structure?

1. **Single source of truth** - Roles are defined once
2. **Type safety** - Invalid roles/actions caught at compile time
3. **Easy to audit** - Just look at this file to see all permissions
4. **Easy to modify** - Add a role or permission in one place

## What's Missing

This simple RBAC system doesn't yet handle:

- What **resources** can be accessed (all documents? just your own?)
- What **fields** can be read or written
- Any **conditions** on the permissions

We'll add those with ABAC later. For now, let's use this to protect our routes.
