---
description: "Create a modal form for adding and editing permission rules with dropdowns and checkboxes."
---

# Rule Editor Modal

A visual form for creating and editing permission rules.

## Live Code: Rule Editor Component

```tsx
// client/src/admin/RuleEditor.tsx

interface RuleEditorProps {
  rule?: PermissionRule
  onSave: (rule: Partial<PermissionRule>) => void
  onClose: () => void
}

const ROLES = ["viewer", "editor", "admin"]
const ACTIONS = ["read", "create", "update", "delete", "manage"]
const SUBJECTS = ["Document", "User", "Comment"]
const DOCUMENT_FIELDS = [
  "id",
  "title",
  "content",
  "status",
  "authorId",
  "orgId",
  "internalNotes",
  "createdAt",
]

export function RuleEditor({ rule, onSave, onClose }: RuleEditorProps) {
  const [form, setForm] = useState({
    type: rule?.type || "allow",
    role: rule?.role || "viewer",
    action: rule?.action || "read",
    subject: rule?.subject || "Document",
    fields: rule?.fields || [],
    conditionField: "",
    conditionValue: "",
  })

  return (
    <div className="modal">
      <h3>{rule ? "Edit Rule" : "Add Rule"}</h3>

      <label>
        Type:
        <select
          value={form.type}
          onChange={(e) => setForm({ ...form, type: e.target.value })}
        >
          <option value="allow">Allow</option>
          <option value="deny">Deny</option>
        </select>
      </label>

      <label>
        Role:
        <select
          value={form.role}
          onChange={(e) => setForm({ ...form, role: e.target.value })}
        >
          {ROLES.map((r) => (
            <option key={r} value={r}>
              {r}
            </option>
          ))}
        </select>
      </label>

      <label>
        Action:
        <select
          value={form.action}
          onChange={(e) => setForm({ ...form, action: e.target.value })}
        >
          {ACTIONS.map((a) => (
            <option key={a} value={a}>
              {a}
            </option>
          ))}
        </select>
      </label>

      <label>
        Subject:
        <select
          value={form.subject}
          onChange={(e) => setForm({ ...form, subject: e.target.value })}
        >
          {SUBJECTS.map((s) => (
            <option key={s} value={s}>
              {s}
            </option>
          ))}
        </select>
      </label>

      <fieldset>
        <legend>Fields (leave empty for all)</legend>
        {DOCUMENT_FIELDS.map((field) => (
          <label key={field}>
            <input
              type="checkbox"
              checked={form.fields.includes(field)}
              onChange={(e) => {
                const fields = e.target.checked
                  ? [...form.fields, field]
                  : form.fields.filter((f) => f !== field)
                setForm({ ...form, fields })
              }}
            />
            {field}
          </label>
        ))}
      </fieldset>

      <fieldset>
        <legend>Condition (optional)</legend>
        <select
          value={form.conditionField}
          onChange={(e) => setForm({ ...form, conditionField: e.target.value })}
        >
          <option value="">No condition</option>
          <option value="authorId">authorId = current user</option>
          <option value="status">status</option>
        </select>
        {form.conditionField === "status" && (
          <input
            placeholder="Value (e.g., draft)"
            value={form.conditionValue}
            onChange={(e) =>
              setForm({ ...form, conditionValue: e.target.value })
            }
          />
        )}
      </fieldset>

      <button onClick={() => onSave(buildRule(form))}>Save</button>
      <button onClick={onClose}>Cancel</button>
    </div>
  )
}
```

This gives non-developers a visual way to create permission rules.
