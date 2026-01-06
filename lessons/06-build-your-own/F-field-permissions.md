---
description: "Implement field-level permission checking and filtering."
---

# Field Permissions

Let's add methods for field-level access control.

## Live Code: Field Methods

```typescript
// Add to PermissionsImpl class

allowedFields(action: string, subject: TSubject): string[] {
  const fields = new Set<string>();

  // Collect fields from all matching allow rules
  const allowRules = this.rules.filter(
    r => r.type === 'allow' &&
         this.matchesAction(r.action, action) &&
         this.matchesSubject(r.subject, subject) &&
         r.fields // Only rules with explicit fields
  );

  for (const rule of allowRules) {
    if (rule.fields) {
      rule.fields.forEach(f => fields.add(f));
    }
  }

  return Array.from(fields);
}

filterFields<T extends Record<string, unknown>>(
  subject: T,
  action: string = 'read'
): Partial<T> {
  const subjectType = this.getSubjectType(subject);
  const allowed = this.allowedFields(action, subjectType);

  // If no field rules defined, return all fields
  if (allowed.length === 0) {
    // Check if we have any allow rule (without fields) for this action
    const hasGeneralAllow = this.rules.some(
      r => r.type === 'allow' &&
           this.matchesAction(r.action, action) &&
           this.matchesSubject(r.subject, subjectType) &&
           !r.fields
    );
    if (hasGeneralAllow) return subject;
    return {} as Partial<T>;
  }

  const filtered: Partial<T> = {};
  for (const field of allowed) {
    if (field in subject) {
      (filtered as any)[field] = subject[field];
    }
  }

  return filtered;
}
```

## Usage

```typescript
const permissions = createPermissions((allow, deny, { user }) => {
  // All users can read basic fields
  allow("read", "Document", ["id", "title", "content", "status"])

  // Editors can also see authorId
  if (user.role === "editor" || user.role === "admin") {
    allow("read", "Document", ["authorId"])
  }

  // Only admins can see internal notes
  if (user.role === "admin") {
    allow("read", "Document", ["internalNotes", "orgId"])
  }
}, currentUser)

// Get all readable fields
permissions.allowedFields("read", "Document")
// Viewer: ['id', 'title', 'content', 'status']
// Editor: ['id', 'title', 'content', 'status', 'authorId']
// Admin:  ['id', 'title', 'content', 'status', 'authorId', 'internalNotes', 'orgId']

// Filter a document object
const doc = { id: 1, title: "Hi", internalNotes: "Secret" }
permissions.filterFields(doc)
// Viewer gets: { id: 1, title: 'Hi' }
// Admin gets:  { id: 1, title: 'Hi', internalNotes: 'Secret' }
```
