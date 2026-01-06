---
description: "Build Express middleware that checks user roles before allowing access to protected routes."
---

# Backend Middleware

Now let's protect our API routes. We'll create middleware that checks if the user's role has the required permission.

## Live Code: Permission Middleware

Create `server/src/middleware/auth.ts`:

```typescript
// server/src/middleware/auth.ts

import { Request, Response, NextFunction } from "express"
import { hasPermission, type Action, type Role } from "../permissions/roles"

// Extend Express Request to include our user
declare global {
  namespace Express {
    interface Request {
      user?: {
        id: number
        email: string
        name: string
        role: Role
        orgId: number
      }
    }
  }
}

// Middleware to require authentication
export function requireAuth(req: Request, res: Response, next: NextFunction) {
  if (!req.user) {
    return res.status(401).json({ error: "Authentication required" })
  }
  next()
}

// Middleware to require a specific permission
export function requirePermission(action: Action) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Authentication required" })
    }

    if (!hasPermission(req.user.role, action)) {
      return res.status(403).json({
        error: "Forbidden",
        message: `You do not have permission to ${action}`,
      })
    }

    next()
  }
}

// Middleware to require a minimum role level
export function requireRole(minimumRole: Role) {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: "Authentication required" })
    }

    const hierarchy: Role[] = ["viewer", "editor", "admin"]
    const userLevel = hierarchy.indexOf(req.user.role)
    const requiredLevel = hierarchy.indexOf(minimumRole)

    if (userLevel < requiredLevel) {
      return res.status(403).json({
        error: "Forbidden",
        message: `This action requires ${minimumRole} role or higher`,
      })
    }

    next()
  }
}
```

## Applying Middleware to Routes

Update your routes to use the middleware:

```typescript
// server/src/routes/documents.ts

import { Router } from "express"
import { requireAuth, requirePermission } from "../middleware/auth"

const router = Router()

// All document routes require authentication
router.use(requireAuth)

// GET /api/documents - anyone authenticated can read
router.get("/", requirePermission("read"), async (req, res) => {
  const documents = await db.select().from(documentsTable)
  res.json(documents)
})

// POST /api/documents - requires create permission
router.post("/", requirePermission("create"), async (req, res) => {
  const { title, content } = req.body
  const document = await db
    .insert(documentsTable)
    .values({
      title,
      content,
      authorId: req.user!.id,
      orgId: req.user!.orgId,
    })
    .returning()
  res.status(201).json(document[0])
})

// PUT /api/documents/:id - requires update permission
router.put("/:id", requirePermission("update"), async (req, res) => {
  const { id } = req.params
  const { title, content } = req.body

  const document = await db
    .update(documentsTable)
    .set({ title, content })
    .where(eq(documentsTable.id, parseInt(id)))
    .returning()

  res.json(document[0])
})

// DELETE /api/documents/:id - requires delete permission
router.delete("/:id", requirePermission("delete"), async (req, res) => {
  const { id } = req.params

  await db.delete(documentsTable).where(eq(documentsTable.id, parseInt(id)))

  res.status(204).send()
})

export default router
```

## Testing the Middleware

Let's verify our protection works:

### As a Viewer

```bash
# Login as viewer
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"viewer@acme.com","password":"password"}'

# Try to create a document (should fail)
curl -X POST http://localhost:3001/api/documents \
  -H "Content-Type: application/json" \
  -H "Cookie: session=..." \
  -d '{"title":"Test","content":"Hello"}'

# Response: 403 Forbidden
# {"error":"Forbidden","message":"You do not have permission to create"}
```

### As an Editor

```bash
# Login as editor
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"editor@acme.com","password":"password"}'

# Create a document (should succeed)
curl -X POST http://localhost:3001/api/documents \
  -H "Content-Type: application/json" \
  -H "Cookie: session=..." \
  -d '{"title":"Test","content":"Hello"}'

# Response: 201 Created

# Try to delete (should fail)
curl -X DELETE http://localhost:3001/api/documents/1 \
  -H "Cookie: session=..."

# Response: 403 Forbidden
```

## What We've Achieved

✅ Unauthenticated users can't access protected routes (401)
✅ Viewers can only read (403 on create/update/delete)
✅ Editors can read, create, and update (403 on delete)
✅ Admins can do everything

## What's Still Wrong

But wait—there's still a problem. Try this as an editor:

1. Log in as `editor@acme.com`
2. Get the list of documents
3. Edit a document that belongs to `admin@acme.com`

**It works!** 😱

Editors can edit ANY document, not just their own. We need ABAC to fix this. But first, let's update the frontend.
