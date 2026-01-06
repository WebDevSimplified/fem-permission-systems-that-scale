---
description: "Tour the Document Management System starter app and understand why it currently has no security - anyone can do anything."
---

# Starter App Tour

Let's explore the application we'll be adding permissions to throughout this workshop.

## The Document Management System

We have a simple but realistic application: a **Document Management System** for organizations. It has:

- **Users** with different roles (admin, editor, viewer)
- **Organizations** (multi-tenant)
- **Documents** with various fields and statuses
- **Comments** on documents

## Tech Stack

- **Backend:** Express + TypeScript
- **Database:** SQLite + Drizzle ORM
- **Frontend:** React + TypeScript (Vite)
- **Auth:** Session-based with cookies

## Pre-Seeded Accounts

The app comes with test accounts you can log in with:

| Email               | Password   | Role   | Organization |
| ------------------- | ---------- | ------ | ------------ |
| `admin@acme.com`    | `password` | Admin  | Acme Corp    |
| `editor@acme.com`   | `password` | Editor | Acme Corp    |
| `viewer@acme.com`   | `password` | Viewer | Acme Corp    |
| `editor@globex.com` | `password` | Editor | Globex Inc   |
| `admin@globex.com`  | `password` | Admin  | Globex Inc   |

## Live Code: Starting the App

Let's start the application and explore what's there:

```bash
# Install dependencies
npm install

# Start the development servers
npm run dev
```

This starts both the backend API (port 3001) and frontend (port 5173).

## The Current Problem: No Permissions!

Right now, the app has **zero permission checks**. Let's see why that's dangerous:

### Problem 1: Anyone Can Edit Anything

1. Log in as `viewer@acme.com`
2. Navigate to any document
3. Click "Edit" - it works!

Viewers shouldn't be able to edit documents, but there's no check preventing it.

### Problem 2: Cross-Organization Access

1. Log in as `editor@acme.com`
2. You can see and edit documents belonging to Globex Inc

Organizations should be isolated from each other.

### Problem 3: Sensitive Fields Exposed

1. Log in as `viewer@acme.com`
2. Look at any document's details
3. You can see `internalNotes` - a field meant only for admins!

### Problem 4: Anyone Can Delete

1. Log in as `editor@acme.com`
2. Click "Delete" on any document - it works!

Only admins should be able to delete documents.

## Document Fields

Here are the fields on a document, and who _should_ be able to access them:

| Field           | Viewer Read | Editor Read | Admin Read | Editor Write | Admin Write |
| --------------- | ----------- | ----------- | ---------- | ------------ | ----------- |
| `id`            | ✅          | ✅          | ✅         | ❌           | ❌          |
| `title`         | ✅          | ✅          | ✅         | ✅           | ✅          |
| `content`       | ✅          | ✅          | ✅         | ✅           | ✅          |
| `status`        | ✅          | ✅          | ✅         | ❌           | ✅          |
| `authorId`      | ❌          | ✅          | ✅         | ❌           | ✅          |
| `orgId`         | ❌          | ❌          | ✅         | ❌           | ✅          |
| `internalNotes` | ❌          | ❌          | ✅         | ❌           | ✅          |
| `createdAt`     | ✅          | ✅          | ✅         | ❌           | ❌          |

Currently, everyone sees everything and can edit everything. We'll fix that.

## The API Routes

Here are the routes we need to protect:

```text
GET    /api/documents          - List all documents
POST   /api/documents          - Create a document
GET    /api/documents/:id      - Get a single document
PUT    /api/documents/:id      - Update a document
DELETE /api/documents/:id      - Delete a document

GET    /api/documents/:id/comments    - List comments
POST   /api/documents/:id/comments    - Create a comment
DELETE /api/comments/:id              - Delete a comment
```

## Our Goal

By the end of this workshop, we'll have:

1. ✅ Role-based route protection (RBAC)
2. ✅ Ownership-based access (only edit your own documents)
3. ✅ Organization isolation (can't see other orgs' data)
4. ✅ Field-level read permissions (hide sensitive fields)
5. ✅ Field-level write permissions (prevent editing certain fields)
6. ✅ A reusable permission library
7. ✅ React components for UI permission checks

Let's start by adding basic role-based access control.
