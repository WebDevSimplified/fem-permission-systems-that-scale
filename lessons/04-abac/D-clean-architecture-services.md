---
title: "Clean Architecture: The Services Layer"
---

Our ABAC system works, but authorization logic is scattered everywhere. Let's fix that by introducing a **services layer** that centralizes all permission checks.

## The Problem

Currently, auth checks live in multiple places:

```text
📁 src/
├── 📁 actions/
│   ├── documents.ts    ← Auth checks here
│   └── projects.ts     ← Auth checks here
├── 📁 dal/
│   └── documents.ts    ← And here
└── 📁 app/
    └── (pages)/
        ├── documents/  ← And here too!
        └── projects/
```

This creates problems:

1. **Duplication** - Same checks written multiple ways
2. **Inconsistency** - Easy to forget a check somewhere
3. **Difficult testing** - Auth logic tied to framework code
4. **Performance** - Potentially extra database queries

## The Solution: Services Layer

We'll create a dedicated layer for business logic which includes authorization checks:

```text
📁 src/
├── 📁 services/        ← NEW: All auth lives here
│   ├── documents.ts
│   └── projects.ts
├── 📁 actions/         ← Just validates & calls services
├── 📁 dal/             ← Just database access
└── 📁 app/             ← Just UI concerns
```

## A Basic Service

The key to our service layer is that each function in this layer should take into account all authorization checks and only return data the user has access to. This centralizes authorization logic and ensures consistency across the application.

```typescript
export async function createDocumentService(
  projectId: string,
  data: DocumentFormValues,
) {
  const user = await getCurrentUser()
  if (user == null) throw new Error("Unauthenticated")

  const permissions = await getUserPermissions()
  if (!permissions.can("document", "create")) throw new AuthorizationError()

  const restrictedData = permissions.pickPermittedFields(
    "document",
    "create",
    data,
  )
  const result = documentSchema.safeParse(restrictedData)
  if (!result.success) throw new Error("Invalid data")

  return await createDocument({
    ...result.data,
    creatorId: user.id,
    lastEditedById: user.id,
    projectId,
  })
}
```

In the above example we are checking the user's permission to create a document and ensuring only the fields they are allowed to modify are used for the creation. This centralizes authorization logic and prevents unauthorized field modifications.

Our action is now a simple wrapper that calls this service function and handles any errors or responses appropriately:

```typescript
export async function createDocumentAction(
  projectId: string,
  data: DocumentFormValues,
) {
  const [error, document] = await tryFn(() =>
    createDocumentService(projectId, data),
  )

  if (error) return error

  redirect(`/projects/${projectId}/documents/${document.id}`)
}
```

Notice what's **not** in the action:

- No `if (!canUpdate)` checks
- No manual field filtering
- No permission imports

All that complexity lives in the service.

## Refactoring Page Components

Pages delegate to services and no longer include any direct permission checks:

```typescript
export default async function ProjectDocumentsPage({
  params,
}: PageProps<"/projects/[projectId]">) {
  const { projectId } = await params

  // Returns null if not found or not authorized
  const project = await getProjectByIdService(projectId)
  if (project == null) return notFound()

  // Only fetch documents the user is authorized to see
  const documents = await getProjectDocumentsService(projectId)

  return (
    // UI
  )
}
```

## Architecture Benefits

### 1. Single Responsibility

| Layer        | Responsibility                          |
| ------------ | --------------------------------------- |
| **Pages**    | Routing, layout, calling services       |
| **Actions**  | Form handling, validation, revalidation |
| **Services** | Business logic, authorization           |
| **DAL**      | Pure database operations                |

This is very important since now you can use the data access layer to get data without having to worry about authorization logic or user context since many times in your application you may want to get all records regardless of the current user's permissions for specific business logic.

### 2. Testability

Since our data access layer, actions, and ui are no longer tied to the user or permissions we can easily test those layers in isolation. It is also easier to test our authorization logic since it all lives in one place.

### 3. Security by Default

As long as all calls go through the service layer it is impossible to accidentally bypass authorization checks or incorrectly code an authorization check since it is all handled automatically by the services layer.

### 4. Efficient Queries

By converting our auth permissions to a database query we are able to leverage the database's query engine to efficiently filter results based on the user's permissions, rather than fetching all data and filtering in application code.

## Converting Permissions to Database Queries

The key insight is that ABAC conditions can become `WHERE` clauses:

```typescript
// Permission condition
{ status: "published" }

// Becomes database query
WHERE status = 'published'

// Multiple conditions from different permissions
{ status: "published" } OR { status: "archived" } OR { creatorId: userId }

// Becomes
WHERE status = 'published' OR status = 'archived' OR creatorId = ?
```

This is powerful because:

- We only fetch data the user can see
- The database does the filtering (efficient!)
- No chance of data leaks from forgetting a filter

## Live Coding Exercise

We'll refactor the codebase to:

1. Create `src/services/documents.ts` and `src/services/projects.ts`
2. Implement `toDrizzleWhere()` for automatic query generation
3. Refactor actions to use services
4. Update page components to use services
5. Remove auth code from actions and DAL

## Branch Checkpoint

After completing this lesson, your code should match:

```bash
git checkout 6-abac-advanced
```

## Summary

The services layer pattern:

- **Centralizes** all authorization in one place
- **Converts** ABAC conditions to database queries
- **Simplifies** actions and components
- **Enables** easy testing of permissions
- **Prevents** data leaks by filtering at the query level

Our ABAC implementation is now:

- ✅ Type-safe
- ✅ Field-level permissions
- ✅ Environment-aware (time-based rules)
- ✅ Clean architecture
- ✅ Database-efficient
