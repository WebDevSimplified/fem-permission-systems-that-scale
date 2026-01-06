---
description: "Add role-based UI controls in React to show, hide, and disable elements based on user permissions."
---

# Frontend Permission Checks

Now let's update the React frontend to show/hide UI elements based on the user's role.

## Live Code: Permission Context

First, let's create a context to share permission info across components:

```typescript
// client/src/context/PermissionContext.tsx

import { createContext, useContext, ReactNode } from "react"
import { useAuth } from "./AuthContext"

type Role = "viewer" | "editor" | "admin"
type Action = "read" | "create" | "update" | "delete" | "manage"

const ROLES: Record<Role, Action[]> = {
  viewer: ["read"],
  editor: ["read", "create", "update"],
  admin: ["read", "create", "update", "delete", "manage"],
}

interface PermissionContextType {
  can: (action: Action) => boolean
  role: Role | null
}

const PermissionContext = createContext<PermissionContextType>({
  can: () => false,
  role: null,
})

export function PermissionProvider({ children }: { children: ReactNode }) {
  const { user } = useAuth()

  const role = user?.role as Role | null

  const can = (action: Action): boolean => {
    if (!role) return false
    return ROLES[role].includes(action)
  }

  return (
    <PermissionContext.Provider value={{ can, role }}>
      {children}
    </PermissionContext.Provider>
  )
}

export function usePermissions() {
  return useContext(PermissionContext)
}
```

## Live Code: Can Component

Create a declarative component for conditional rendering:

```typescript
// client/src/components/Can.tsx

import { ReactNode } from "react"
import { usePermissions } from "../context/PermissionContext"

type Action = "read" | "create" | "update" | "delete" | "manage"

interface CanProps {
  action: Action
  children: ReactNode
  fallback?: ReactNode
}

export function Can({ action, children, fallback = null }: CanProps) {
  const { can } = usePermissions()

  if (can(action)) {
    return <>{children}</>
  }

  return <>{fallback}</>
}
```

## Using in Components

Now update the document list to use permissions:

```tsx
// client/src/pages/DocumentList.tsx

import { Can } from "../components/Can"
import { usePermissions } from "../context/PermissionContext"
import { Link } from "react-router-dom"

export function DocumentList() {
  const { documents } = useDocuments()
  const { can } = usePermissions()

  return (
    <div>
      <h1>Documents</h1>

      {/* Only show Create button if user can create */}
      <Can action="create">
        <Link to="/documents/new">
          <button>Create Document</button>
        </Link>
      </Can>

      <ul>
        {documents.map((doc) => (
          <li key={doc.id}>
            <Link to={`/documents/${doc.id}`}>{doc.title}</Link>

            {/* Only show Edit if user can update */}
            <Can action="update">
              <Link to={`/documents/${doc.id}/edit`}>
                <button>Edit</button>
              </Link>
            </Can>

            {/* Only show Delete if user can delete */}
            <Can action="delete">
              <button onClick={() => handleDelete(doc.id)}>Delete</button>
            </Can>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

## Alternative: Using the Hook Directly

Sometimes you need more control:

```tsx
function DocumentActions({ document }) {
  const { can, role } = usePermissions()

  // Conditional logic based on permissions
  const showAdvancedOptions = role === "admin"

  return (
    <div>
      {can("update") && <button onClick={handleEdit}>Edit</button>}

      {can("delete") && <button onClick={handleDelete}>Delete</button>}

      {showAdvancedOptions && <button onClick={handleArchive}>Archive</button>}
    </div>
  )
}
```

## Disabling vs Hiding

Sometimes you want to show a disabled button instead of hiding it:

```tsx
function DocumentActions({ document }) {
  const { can } = usePermissions()

  return (
    <div>
      {/* Hide if no permission */}
      <Can action="update">
        <button onClick={handleEdit}>Edit</button>
      </Can>

      {/* Show but disable if no permission */}
      <button
        onClick={handleDelete}
        disabled={!can("delete")}
        title={!can("delete") ? "Only admins can delete" : undefined}
      >
        Delete
      </button>
    </div>
  )
}
```

**When to hide vs disable:**

- **Hide** when the action isn't relevant to the user's workflow
- **Disable** when you want users to know the feature exists (encourages upgrades, shows what's possible)

## Testing the UI

1. Log in as `viewer@acme.com` - you should only see "Read" actions
2. Log in as `editor@acme.com` - you should see Edit but not Delete
3. Log in as `admin@acme.com` - you should see everything

## ⚠️ Important Reminder

These UI checks are for **user experience only**. They do NOT provide security!

The backend middleware we created earlier is what actually enforces permissions. The frontend just makes the UI cleaner.

In the next lesson, we'll demonstrate exactly why relying only on frontend checks is dangerous.
