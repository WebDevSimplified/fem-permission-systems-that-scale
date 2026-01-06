---
description: "Define the TypeScript types and interfaces for our permission library."
---

# Core Types

Let's define the type system for our permission library.

## Live Code: Type Definitions

```typescript
// packages/permissions/src/types.ts

// A rule represents a single permission
export interface Rule<TSubject extends string = string> {
  type: "allow" | "deny"
  action: string
  subject: TSubject
  conditions?: Record<string, unknown>
  fields?: string[]
}

// Context passed to permission builder
export interface PermissionContext<TUser> {
  user: TUser
}

// The builder functions passed to createPermissions callback
export type AllowFn<TSubject extends string> = (
  action: string,
  subject: TSubject,
  conditionsOrFields?: Record<string, unknown> | string[],
  fields?: string[]
) => void

export type DenyFn<TSubject extends string> = (
  action: string,
  subject: TSubject,
  conditions?: Record<string, unknown>
) => void

// The callback signature for createPermissions
export type PermissionBuilder<TUser, TSubject extends string> = (
  allow: AllowFn<TSubject>,
  deny: DenyFn<TSubject>,
  context: PermissionContext<TUser>
) => void

// The main Permissions class interface
export interface Permissions<TSubject extends string = string> {
  can(action: string, subject: TSubject | object, field?: string): boolean
  cannot(action: string, subject: TSubject | object, field?: string): boolean
  authorize(action: string, subject: TSubject | object): void
  allowedFields(action: string, subject: TSubject): string[]
  filterFields<T extends Record<string, unknown>>(subject: T): Partial<T>
}
```

## Type Parameters

- `TUser` - Your user type (with role, id, etc.)
- `TSubject` - Union of subject strings (`'Document' | 'User' | 'Comment'`)

This gives us full type safety when defining and checking permissions.
