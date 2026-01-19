---
title: "Upgrading to CASL"
---

Let's upgrade our custom ABAC implementation to use CASL.

## Installing CASL

First, install the CASL packages:

```bash
npm install @casl/ability
```

## Updating the Permission Module

Here's how our `abac.ts` file transforms to use CASL:

### Before (Custom Implementation)

```typescript
class PermissionBuilder {
  // ... custom logic
}

async function getUserPermissionsInternal(user: User) {
  const builder = new PermissionBuilder()

  if (user.role === "admin") {
    builder.can("document", "read")
    builder.can("project", "read")
  }

  if (user.role === "viewer") {
    builder.can("document", "read", { status: "published" }, [
      "title",
      "content",
    ])
  }

  return builder.build()
}
```

### After (CASL)

```typescript
type FullCRUD = "create" | "read" | "update" | "delete"
type ProjectSubject = "project" | Pick<Project, "department">
type DocumentSubject =
  | "document"
  | Pick<Document, "projectId" | "creatorId" | "status" | "isLocked">

type MyAbility = MongoAbility<
  [FullCRUD, ProjectSubject] | [FullCRUD, DocumentSubject]
>

async function getUserPermissionsInternal(user: User) {
  const { can: allow, build } = new AbilityBuilder<MyAbility>(
    createMongoAbility,
  )

  if (user.role === "admin") {
    can("read", "document")
    can("read", "project")
  }

  if (user.role === "viewer") {
    can("read", "document", ["title", "content"], { status: "published" })
  }

  return build()
}
```

## Using CASL in Components

The usage pattern is almost identical:

```typescript
// Before (custom)
const permissions = await getUserPermissions()
if (permissions.can("document", "update", document)) {
  // ...
}

// After (CASL)
import { subject } from "@casl/ability"

const permissions = await getUserPermissions()
if (permissions.can("update", subject("document", document))) {
  // ...
}
```

Note the differences:

1. Resource and action order is swapped: `can("update", "document")` vs `permissions.can("document", "update")`
2. Use `subject()` helper to wrap data for condition checking

## Field-Level Permissions with CASL

CASL has built-in support for field permissions:

```typescript
// Define which fields each permission applies to
can("update", "document", ["title", "content", "status"], { isLocked: false })
can("update", "document", ["title", "content"], { isLocked: false }) // More restricted

// Check field permission
ability.can("update", subject("document", document), "status")
```

## CASL Bonus Features

### 1. Rule Serialization

You can serialize CASL permissions to a JSON object and send them to the client or use them in other parts of your application:

```typescript
// Send ability rules to the client
const permissions = await getUserPermissions()
const rules = permissions.rules

// Recreate ability on client
import { createMongoAbility } from "@casl/ability"
const clientPermissions = createMongoAbility(rules)
```

### 2. Advanced Conditions

CASL allows you to define complex conditions for permissions. For example, you can check not equal, in array, etc.

```typescript
can("read", "document", { status: { $ne: "archived" } })
can("read", "document", { status: { $in: ["published", "draft"] } })
```

## CASL Drawbacks

CASL is great at many things, but since it was created as a backend Node focused project it struggles when used with frontend frameworks like React or Next.js due to its reliance on classes.

### 1. Reliance on Classes

CASL uses class names to determine which permissions to check on an object which is why our permissions do not work by default since we aren't using classes. Instead, we need to wrap everything in a `subject()` call to tell CASL what type our object.

### 2. React Server Component Compatibility

The `subject()` function unfortunately, mutates our objects in a way that is incompatible with React Server components. To get around this you need to spread a new object into `subject()` like `subject("document", { ...document })`. Another option is to use the `detectSubjectType` build option which is what we will be doing.

## Branch Checkpoint

After completing this lesson, your code should match:

```bash
git checkout 7-casl
```

## Summary

Migrating to CASL gives us:

- ✅ Battle-tested permission library
- ✅ Built-in field-level permissions
- ✅ Community support and documentation

The concepts are identical to what we built, but using a library like CASL can give you extra functionality and make it easier to work with on a larger team.
