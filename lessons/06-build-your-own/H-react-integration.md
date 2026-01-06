---
description: "Create React components and hooks for frontend permission checks."
---

# React Integration

Let's create React components for declarative permission checks.

## Live Code: Permission Context

```typescript
// client/src/permissions/PermissionContext.tsx

import { createContext, useContext, ReactNode, useMemo } from "react"
import { createPermissions, Permissions } from "@our-app/permissions"
import { useAuth } from "../auth/AuthContext"

const PermissionContext = createContext<Permissions | null>(null)

export function PermissionProvider({ children }: { children: ReactNode }) {
  const { user } = useAuth()

  const permissions = useMemo(() => {
    if (!user) return null

    return createPermissions((allow, deny, { user: u }) => {
      allow("read", "Document", { orgId: u.orgId })

      if (u.role === "editor") {
        allow("create", "Document")
        allow("update", "Document", { authorId: u.id })
      }

      if (u.role === "admin") {
        allow("manage", "Document", { orgId: u.orgId })
      }
    }, user)
  }, [user])

  return (
    <PermissionContext.Provider value={permissions}>
      {children}
    </PermissionContext.Provider>
  )
}

export function usePermissions() {
  const permissions = useContext(PermissionContext)
  if (!permissions) {
    throw new Error("usePermissions must be used within PermissionProvider")
  }
  return permissions
}
```

## Live Code: Can Component

```tsx
// client/src/permissions/Can.tsx

import { ReactNode } from "react"
import { usePermissions } from "./PermissionContext"

interface CanProps {
  action: string
  subject: string | Record<string, unknown>
  field?: string
  children: ReactNode
  fallback?: ReactNode
}

export function Can({
  action,
  subject,
  field,
  children,
  fallback = null,
}: CanProps) {
  const permissions = usePermissions()

  if (permissions.can(action, subject, field)) {
    return <>{children}</>
  }

  return <>{fallback}</>
}
```

## Usage in Components

```tsx
function DocumentCard({ document }) {
  return (
    <div>
      <h2>{document.title}</h2>

      <Can action="update" subject={document}>
        <button>Edit</button>
      </Can>

      <Can action="delete" subject={document}>
        <button>Delete</button>
      </Can>

      <Can action="read" subject={document} field="internalNotes">
        <p>Notes: {document.internalNotes}</p>
      </Can>
    </div>
  )
}
```
