---
description: "Build a simple visual interface for viewing and editing permission rules."
---

# Visual Editor UI

Let's build a simple admin UI for managing permissions.

## Live Code: Permission List Component

```tsx
// client/src/admin/PermissionList.tsx

import { useState, useEffect } from "react"

interface PermissionRule {
  id: number
  type: "allow" | "deny"
  role: string
  action: string
  subject: string
  conditions?: Record<string, string>
  fields?: string[]
}

export function PermissionList() {
  const [rules, setRules] = useState<PermissionRule[]>([])

  useEffect(() => {
    fetch("/api/admin/permissions")
      .then((r) => r.json())
      .then(setRules)
  }, [])

  return (
    <div className="permission-list">
      <h2>Permission Rules</h2>

      <table>
        <thead>
          <tr>
            <th>Type</th>
            <th>Role</th>
            <th>Action</th>
            <th>Subject</th>
            <th>Conditions</th>
            <th>Fields</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {rules.map((rule) => (
            <tr key={rule.id} className={rule.type}>
              <td>{rule.type}</td>
              <td>{rule.role}</td>
              <td>{rule.action}</td>
              <td>{rule.subject}</td>
              <td>{JSON.stringify(rule.conditions) || "-"}</td>
              <td>{rule.fields?.join(", ") || "all"}</td>
              <td>
                <button onClick={() => handleEdit(rule)}>Edit</button>
                <button onClick={() => handleDelete(rule.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      <button onClick={() => setShowModal(true)}>Add Rule</button>
    </div>
  )
}
```

## Styling

```css
.permission-list tr.allow {
  background: #e6ffe6;
}
.permission-list tr.deny {
  background: #ffe6e6;
}
```

The list shows all rules with visual distinction between allow (green) and deny (red).
