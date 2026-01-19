---
title: "Advanced ABAC Features"
---

Our basic ABAC system works, but real applications need more sophisticated features. Let's add **field-level permissions** and explore how ABAC can express complex policies.

## New Requirements

In many applications certain users are restricted on which fields they can read or modify. This is something that is impossible to do in RBAC, but ABAC makes these policies straightforward to express using field-level permissions.

### 1. Field-Level Read Permissions

Different roles should see different fields on documents:

| Field            | Admin | Editor | Author | Viewer |
| ---------------- | ----- | ------ | ------ | ------ |
| `title`          | ✅    | ✅     | ✅     | ✅     |
| `content`        | ✅    | ✅     | ✅     | ✅     |
| `status`         | ✅    | ✅     | ✅     | ✅     |
| `isLocked`       | ✅    | ✅     | ✅     | ❌     |
| `creatorId`      | ✅    | ✅     | ✅     | ❌     |
| `lastEditedById` | ✅    | ✅     | ✅     | ❌     |
| `createdAt`      | ✅    | ✅     | ✅     | ❌     |
| `updatedAt`      | ✅    | ✅     | ✅     | ❌     |

### 2. Field-Level Write Permissions

When creating or editing documents, only certain fields should be writable by each role:

| Field      | Admin | Editor | Author |
| ---------- | ----- | ------ | ------ |
| `title`    | ✅    | ✅     | ✅     |
| `content`  | ✅    | ✅     | ✅     |
| `status`   | ✅    | ✅     | ❌     |
| `isLocked` | ✅    | ❌     | ❌     |

Authors can write content but can't change document status or lock documents.

### 3. Environment-Based Rules

Since this company values work-life balance, editors and authors are not allowed to make changes on weekends which is an example of an environment-based rule.

## Extending the Permission API

We need to enhance our `can()` function to support field checks:

```typescript
// Check if user can read a specific field
permissions.can("document", "read", document, "isLocked")

// Check if user can update a specific field
permissions.can("document", "update", document, "status")
```

## Enhanced Policy Definitions

The `allow` function now accepts an optional fields array:

```typescript
function addEditorPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department">,
) {
  builder.allow("document", "read") // All fields by default

  builder.allow(
    "document",
    "update",
    { isLocked: false },
    ["content", "title", "status"], // Restricted fields!
  )
}
```

## The Enhanced Permission Builder

```typescript
type Permission<Res extends keyof Resources> = {
  action: Resources[Res]["action"]
  condition?: Partial<Resources[Res]["condition"]>
  fields?: string[] // New: allowed fields
}

class PermissionBuilder {
  // ...existing code...

  allow<Res extends keyof Resources>(
    resource: Res,
    action: Resources[Res]["action"],
    condition?: Partial<Resources[Res]["condition"]>,
    fields?: string[], // New parameter
  ) {
    this.#permissions[resource].push({ action, condition, fields })
    return this
  }

  build() {
    const permissions = this.#permissions

    return {
      can<Res extends keyof Resources>(
        resource: Res,
        action: Resources[Res]["action"],
        data?: Resources[Res]["condition"],
        field?: string, // New: optional field to check
      ) {
        return permissions[resource].some((perm) => {
          if (perm.action !== action) return false

          // Check conditions (existing logic)
          const conditionsPassed =
            perm.condition == null ||
            data == null ||
            Object.entries(perm.condition).every(
              ([key, value]) => data[key as keyof typeof data] === value,
            )

          if (!conditionsPassed) return false

          // Check field permission (new logic)
          if (field != null && perm.fields != null) {
            return perm.fields.includes(field)
          }

          return true
        })
      },
    }
  }
}
```

## Using Field Permissions

Adding support for field permissions takes a little bit of TypeScript wizardry and extra work, but for the most part isn't too bad. Where field permissions become difficult to work with is when consuming which fields a user has access to and ensuring your UI works for all possible permutations.

### In Forms

```tsx
// Edit document page
const permissions = getUserPermissions(user)

<DocumentForm
  document={document}
  projectId={projectId}
  canModify={{
    status: permissions.can("document", "update", document, "status"),
    isLocked: permissions.can("document", "update", document, "isLocked"),
  }}
/>
```

### In the Form Component

```tsx
function DocumentForm({ document, projectId, canModify }) {
  return (
    <Form>
      <Input name="title" />
      <Textarea name="content" />

      {/* Only render if user can modify */}
      {canModify.status && (
        <Select name="status">
          <option value="draft">Draft</option>
          <option value="published">Published</option>
          <option value="archived">Archived</option>
        </Select>
      )}

      {canModify.isLocked && <Checkbox name="isLocked" label="Lock document" />}
    </Form>
  )
}
```

### In Display Components

```typescript
// Document detail page
const permissions = getUserPermissions(user)

const canReadField = {
  isLocked: permissions.can("document", "read", document, "isLocked"),
  creator: permissions.can("document", "read", document, "creatorId"),
  lastEditedBy: permissions.can("document", "read", document, "lastEditedById"),
  createdAt: permissions.can("document", "read", document, "createdAt"),
  updatedAt: permissions.can("document", "read", document, "updatedAt"),
}

// In JSX
{canReadField.isLocked && document.isLocked && (
  <Badge>Locked</Badge>
)}

{canReadField.creator && (
  <div>Created by: {document.creator.name}</div>
)}
```

## Filtering Permitted Fields

We can also create a helper to filter data based on permissions:

```typescript
function pickPermittedFields<T extends Record<string, unknown>>(
  permissions: ReturnType<typeof getUserPermissions>,
  resource: keyof Resources,
  action: string,
  data: T,
  resourceData?: Resources[keyof Resources]["condition"],
): Partial<T> {
  const result: Record<string, unknown> = {}

  for (const [key, value] of Object.entries(data)) {
    if (permissions.can(resource, action, resourceData, key)) {
      result[key] = value
    }
  }

  return result as Partial<T>
}

// Usage in a mutation
const restrictedData = pickPermittedFields(
  permissions,
  "document",
  "update",
  formData,
  existingDocument,
)
```

This ensures users can only modify fields they have permission to change, even if they try to send unauthorized fields from the client.

## The Power of ABAC

Notice what we can now express:

```typescript
// "Authors can read the createdAt field only on documents they created"
permissions.can("document", "read", { creatorId: user.id }, "createdAt")

// "Editors can update status on unlocked documents"
permissions.can("document", "update", { isLocked: false }, "status")
```

These complex, conditional, field-level permissions are natural in ABAC but would require an explosion of permission strings in RBAC.

## What's Next

Our ABAC system is powerful, but we still have permission checks scattered throughout:

- Page components
- Server actions
- Mutations
- Query functions

In the next lesson, we'll create a **services layer** that centralizes all authorization logic and even converts permissions to database queries automatically.
