---
description: "Best practices for implementing permission checks in React applications."
---

# Frontend Patterns

Let's review best practices for frontend permission handling.

## Pattern 1: Hide vs Disable

```tsx
// HIDE - User doesn't know feature exists
<Can action="delete" subject={document}>
  <button>Delete</button>
</Can>

// DISABLE - User knows feature exists but can't use it
<button
  disabled={!permissions.can('delete', document)}
  title={!permissions.can('delete', document) ? 'Admin only' : undefined}
>
  Delete
</button>
```

**When to hide:** Feature isn't relevant to user's workflow
**When to disable:** Show capability exists (drives upgrades, reduces confusion)

## Pattern 2: Route Guards

```tsx
// ProtectedRoute.tsx
function ProtectedRoute({ action, subject, children }) {
  const permissions = usePermissions()

  if (!permissions.can(action, subject)) {
    return <Navigate to="/unauthorized" />
  }

  return children
}

// Usage
;<Route
  path="/admin"
  element={
    <ProtectedRoute action="manage" subject="User">
      <AdminPanel />
    </ProtectedRoute>
  }
/>
```

## Pattern 3: Form Field Permissions

```tsx
function DocumentForm({ document }) {
  const permissions = usePermissions()
  const canEditStatus = permissions.can("write", document, "status")

  return (
    <form>
      <input name="title" defaultValue={document.title} />

      <select
        name="status"
        disabled={!canEditStatus}
        defaultValue={document.status}
      >
        <option value="draft">Draft</option>
        <option value="published">Published</option>
      </select>

      {!canEditStatus && (
        <span className="hint">Only admins can change status</span>
      )}
    </form>
  )
}
```

## Pattern 4: Conditional Data Display

```tsx
function DocumentDetails({ document }) {
  const permissions = usePermissions()

  return (
    <dl>
      <dt>Title</dt>
      <dd>{document.title}</dd>

      <Can action="read" subject={document} field="authorId">
        <dt>Author</dt>
        <dd>{document.authorId}</dd>
      </Can>

      <Can action="read" subject={document} field="internalNotes">
        <dt>Internal Notes</dt>
        <dd>{document.internalNotes}</dd>
      </Can>
    </dl>
  )
}
```

Remember: Frontend checks are for UX, not security!
