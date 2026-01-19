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

When creating or editing documents, only certain fields should be writable:

| Field      | Admin | Editor | Author |
| ---------- | ----- | ------ | ------ |
| `title`    | ✅    | ✅     | ✅     |
| `content`  | ✅    | ✅     | ✅     |
| `status`   | ✅    | ✅     | ❌     |
| `isLocked` | ✅    | ❌     | ❌     |

Authors can write content but can't change document status or lock documents.

### 3. 3. Environment-Based Rules

Since this company values work-life balance, editors and authors are not allowed to make changes on weekends which is an example of an **environment-based** rule that uses attributes from neither the user nor the resource.

## Extending the Permission API

We need to enhance our `can()` function to support field checks:

```typescript
// Check if user can read a specific field
permissions.can("document", "read", document, "isLocked")

// Check if user can update a specific field
permissions.can("document", "update", document, "status")
```

## Enhanced Policy Definitions

The `allow` function now accepts an optional fields array as a fourth parameter:

```typescript
function addEditorPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department">,
) {
  builder.allow("document", "read") // All fields by default

  builder.allow(
    "document",
    "update",
    { isLocked: false }, // Condition: only unlocked docs
    ["content", "title", "status"], // Restricted to these fields
  )
}
```

## Updating the Permission Builder

We only need to change two things in our existing builder:

### 1. Add `fields` to the `allow()` method

```typescript
allow<Res extends keyof Resources>(
  resource: Res,
  action: Resources[Res]["action"],
  condition?: Partial<Resources[Res]["condition"]>,
  fields?: string[], // NEW: optional field restriction
) {
  this.#permissions[resource].push({ action, condition, fields })
  return this
}
```

### 2. Check fields in `can()`

Add this after the condition check passes:

```typescript
// Check field permission (new logic)
if (field != null && perm.fields != null) {
  return perm.fields.includes(field)
}
```

The full logic: if a field is requested AND this permission specifies allowed fields, check if the field is in the list. If `perm.fields` is `null`, all fields are allowed.

## Using Field Permissions

Adding support for field permissions takes a little bit of TypeScript wizardry and extra work, but for the most part isn't too bad. Where field permissions become difficult to work with is when consuming which fields a user has access to and ensuring your UI works for all possible permutations.

### In Forms

We compute which fields the user can modify, then pass that info to the form component:

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

Why pass `canModify` as a prop instead of checking permissions inside the form? This keeps the form component **pure** since it just renders based on props, making it easier to test and reuse. It also ensures these components are easier to test and reuse.

### In the Form Component

The form conditionally renders fields based on permissions:

```tsx
function DocumentForm({ document, projectId, canModify }) {
  return (
    <Form>
      <Input name="title" />
      <Textarea name="content" />

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

For read permissions, we hide fields the user can't see:

```typescript
// Document detail page
const permissions = getUserPermissions(user)

const canReadField = {
  isLocked: permissions.can("document", "read", document, "isLocked"),
  creator: permissions.can("document", "read", document, "creatorId"),
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

When handling form submissions, we need to filter out fields the user isn't allowed to modify, even if they try to send them from the client:

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
```

This is useful during mutations since it ensures security even if someone bypasses the UI:

```typescript
const restrictedData = pickPermittedFields(
  permissions,
  "document",
  "update",
  formData,
  existingDocument,
)
// restrictedData only contains fields the user can actually modify
```

## Implementing Environment Rules

For weekend restrictions, we check the current day when building permissions:

```typescript
function addEditorPermissions(
  builder: PermissionBuilder,
  user: Pick<User, "department">,
) {
  const dayOfWeek = new Date().getDay()
  const isWeekend = dayOfWeek === 0 || dayOfWeek === 6

  builder
    .allow("project", "read", { department: user.department })
    .allow("project", "read", { department: null })
    .allow("document", "read")

  // Only allow updates on weekdays
  if (!isWeekend) {
    builder.allow("document", "update", { isLocked: false })
  }
}
```

The permission decision can incorporate **any** attribute: user, resource, or environment.

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
