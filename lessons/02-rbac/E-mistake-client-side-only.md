---
description: "See what happens when you only check permissions on the frontend - and why backend validation is essential."
---

# 🔴 Mistake: Client-Side Only Checks

This is one of the most common security mistakes in web development. Let's see it in action.

## The Scenario

Imagine we "protected" our app by only hiding the UI buttons for unauthorized actions, but forgot to add backend checks.

## Live Demo: Bypassing Frontend Protection

### Step 1: Log in as a Viewer

Log in with `viewer@acme.com` / `password`.

Notice that the Edit and Delete buttons are hidden. The UI looks secure!

### Step 2: Open Developer Tools

Press F12 to open Chrome DevTools, go to the Network tab.

### Step 3: Find an Edit Request

Click on a document to view it. Look at the network request that fetched the document. Note the document ID (e.g., `/api/documents/1`).

### Step 4: Craft a Malicious Request

In the DevTools Console, paste this:

```javascript
// As a viewer, try to edit a document directly
fetch("/api/documents/1", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  credentials: "include", // Include our session cookie
  body: JSON.stringify({
    title: "HACKED BY VIEWER!",
    content: "This should not be possible.",
  }),
})
  .then((r) => r.json())
  .then(console.log)
  .catch(console.error)
```

### Without Backend Protection

If we ONLY had frontend checks, this would succeed! The document would be updated.

**Result:** `{ id: 1, title: "HACKED BY VIEWER!", ... }`

### With Our Backend Middleware

Because we added `requirePermission('update')` middleware, we get:

**Result:** `{ error: "Forbidden", message: "You do not have permission to update" }`

## Why This Matters

Attackers don't use your UI. They can:

- Use browser DevTools
- Use `curl` or Postman
- Write scripts to make requests
- Intercept and modify requests with proxy tools

**Any security check that only exists in the frontend can be bypassed.**

## The Right Approach

Always implement **defense in depth**:

```text
┌─────────────────────────────────────────────────────┐
│  Frontend (UX only)                                 │
│  - Hide buttons user shouldn't click                │
│  - Disable form fields user can't edit              │
│  - Show helpful messages about what's allowed       │
├─────────────────────────────────────────────────────┤
│  Backend (Actual Security)                          │
│  - Validate every request                           │
│  - Check permissions before any action              │
│  - Return 403 if not authorized                     │
├─────────────────────────────────────────────────────┤
│  Database (Additional Layer)                        │
│  - Row-level security policies                      │
│  - Constraints and triggers                         │
│  - Audit logging                                    │
└─────────────────────────────────────────────────────┘
```

## Red Flags to Watch For

When reviewing code, watch for these patterns:

```typescript
// ❌ BAD: Only checking on frontend
function EditButton({ document }) {
  if (user.role !== "editor" && user.role !== "admin") {
    return null // Just hiding the button
  }
  return <button onClick={handleEdit}>Edit</button>
}

// Meanwhile, the backend has no check:
router.put("/documents/:id", async (req, res) => {
  // No permission check here!
  await db
    .update(documents)
    .set(req.body)
    .where(eq(documents.id, req.params.id))
  res.json({ success: true })
})
```

```typescript
// ✅ GOOD: Backend always validates
router.put("/documents/:id", requirePermission("update"), async (req, res) => {
  // Permission checked by middleware before we get here
  await db
    .update(documents)
    .set(req.body)
    .where(eq(documents.id, req.params.id))
  res.json({ success: true })
})
```

## Key Takeaway

> **Frontend checks = User Experience** > **Backend checks = Security**

Never confuse the two. The frontend makes authorized actions convenient. The backend makes unauthorized actions impossible.

## What's Next

Our RBAC system now properly protects routes by role. But there's still a problem: an editor can edit ANY document, not just their own.

Let's add attribute-based checks to fix that.
