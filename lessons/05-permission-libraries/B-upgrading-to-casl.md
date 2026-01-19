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

### Key Changes

Here's what changes when migrating to CASL:

```typescript
// BEFORE (custom): resource first, action second
builder.can("document", "read")
builder.can("document", "read", { status: "published" }, ["title", "content"])

// AFTER (CASL): action first, resource second, fields before conditions
can("read", "document")
can("read", "document", ["title", "content"], { status: "published" })
```

### Type Setup

CASL needs types to understand your resources which are broken down into subjects and actions:

```typescript
type FullCRUD = "create" | "read" | "update" | "delete"
type ProjectSubject = "project" | Pick<Project, "department">
type DocumentSubject =
  | "document"
  | Pick<Document, "projectId" | "creatorId" | "status" | "isLocked">

type MyAbility = MongoAbility<
  [FullCRUD, ProjectSubject] | [FullCRUD, DocumentSubject]
>
```

### Building Permissions

```typescript
async function getUserPermissionsInternal(user: User) {
  const { can: allow, build } = new AbilityBuilder<MyAbility>(
    createMongoAbility,
  )

  if (user.role === "admin") {
    allow("read", "document")
    allow("read", "project")
  }

  if (user.role === "viewer") {
    // Fields come BEFORE conditions in CASL
    allow("read", "document", ["title", "content"], { status: "published" })
  }

  return build()
}
```

## Using CASL in Components

The usage pattern is almost identical, with one key difference:

```typescript
// Custom: pass object directly
if (permissions.can("document", "update", document)) {
  // ...
}

// CASL: wrap object with subject() helper
import { subject } from "@casl/ability"

if (ability.can("update", subject("document", document))) {
  // ...
}
```

Note the differences:

1. Resource and action order is swapped: `can("update", "document")` vs `permissions.can("document", "update")`
2. Use `subject()` helper to wrap data for condition checking

> Why the `subject()` wrapper? CASL needs to know what _type_ of object you're checking. Our custom implementation knew because we passed the resource name first. CASL uses class names by default, but since we're using plain objects, we need to tell it explicitly.

## Field-Level Permissions with CASL

CASL has built-in support for field permissions:

```typescript
// Define which fields each permission applies to
can("update", "document", ["title", "content", "status"], { isLocked: false })

// Check field permission
ability.can("update", subject("document", document), "status")
```

## CASL Bonus Features

### 1. Rule Serialization

You can serialize CASL permissions to JSON and send them to the client:

```typescript
// Server: send ability rules to the client
const permissions = await getUserPermissions()
const rules = permissions.rules

// Client: recreate ability from rules
import { createMongoAbility } from "@casl/ability"
const clientPermissions = createMongoAbility(rules)
```

This enables client-side permission checks without an API call.

### 2. Advanced Conditions

CASL supports MongoDB-style query operators:

```typescript
can("read", "document", { status: { $ne: "archived" } }) // not equal
can("read", "document", { status: { $in: ["published", "draft"] } }) // in array
```

## CASL Drawbacks

CASL is great at many things, but since it was created as a backend Node focused project it struggles when used with frontend frameworks like React or Next.js due to its reliance on classes.

### 1. Reliance on Classes

CASL uses class names to determine which permissions to check on an object. Since we're using plain objects (not class instances), permissions don't work by default which is why we need the `subject()` wrapper everywhere.

### 2. React Server Component Compatibility

The `subject()` function unfortunately, mutates our objects in a way that is incompatible with React Server components. There are two workarounds:

**Option A: Spread into a new object**

```typescript
// Always create a new object before passing to subject()
ability.can("update", subject("document", { ...document }))
```

**Option B: Use `detectSubjectType`**

Configure CASL to detect subject types without mutation:

```typescript
const ability = createMongoAbility(rules, {
  detectSubjectType: (object) => {
    // Check for a type property we add ourselves
    if (object && typeof object === "object" && "__typename" in object) {
      return object.__typename as string
    }
    return undefined
  },
})

// In your queries, add __typename to returned objects
const document = await db.query.documents.findFirst({
  where: eq(documents.id, id),
})

return { ...document, __typename: "document" as const }
```

This approach adds a small overhead but makes consuming CASL permissions easier.

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
