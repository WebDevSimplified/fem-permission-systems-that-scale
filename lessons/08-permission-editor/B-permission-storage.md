---
description: "Create a database schema to store permission rules that can be edited at runtime."
---

# Permission Storage Schema

Let's store permission rules in the database.

## Live Code: Schema Definition

```typescript
// server/src/db/schema.ts

export const permissionRulesTable = sqliteTable("permission_rules", {
  id: integer("id").primaryKey({ autoIncrement: true }),

  // Rule type
  type: text("type", { enum: ["allow", "deny"] }).notNull(),

  // What role this applies to
  role: text("role", { enum: ["viewer", "editor", "admin"] }).notNull(),

  // Action and subject
  action: text("action").notNull(), // 'read', 'create', 'update', 'delete', 'manage'
  subject: text("subject").notNull(), // 'Document', 'User', 'Comment'

  // Conditions as JSON (e.g., { "status": "draft" })
  conditions: text("conditions", { mode: "json" }),

  // Allowed fields as JSON array (e.g., ["title", "content"])
  fields: text("fields", { mode: "json" }),

  // Metadata
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .default(sql`(unixepoch())`),
  createdBy: integer("created_by").references(() => usersTable.id),
})
```

## Live Code: Migration

```bash
npm run db:generate
npm run db:migrate
```

## Seed Default Rules

```typescript
// server/src/db/seed.ts

const defaultRules = [
  // Viewers
  {
    type: "allow",
    role: "viewer",
    action: "read",
    subject: "Document",
    fields: ["id", "title", "content", "status", "createdAt"],
  },

  // Editors
  {
    type: "allow",
    role: "editor",
    action: "read",
    subject: "Document",
    fields: ["id", "title", "content", "status", "createdAt", "authorId"],
  },
  { type: "allow", role: "editor", action: "create", subject: "Document" },
  {
    type: "allow",
    role: "editor",
    action: "update",
    subject: "Document",
    conditions: { authorId: "$user.id" },
  }, // Special placeholder

  // Admins
  { type: "allow", role: "admin", action: "manage", subject: "Document" },
  { type: "allow", role: "admin", action: "manage", subject: "User" },
]

await db.insert(permissionRulesTable).values(defaultRules)
```

Now permissions can be queried and modified at runtime.
