---
description: "Understand the limitations of pure RBAC and when you need to add attribute-based rules."
---

# RBAC Limitations

We've implemented a working RBAC system. But let's see where it falls short.

## The Problem: Coarse-Grained Access

### Scenario 1: Editors Can Edit Everything

Log in as `editor@acme.com` and try this:

1. Go to the document list
2. Find a document authored by `admin@acme.com`
3. Click Edit
4. Change the title and save

**It works!** But it shouldn't. Editors should only edit their own documents.

### Scenario 2: Cross-Organization Access

Log in as `editor@acme.com` (Acme Corp) and:

1. Look at the document list
2. You can see documents from Globex Inc!

**Organizations should be isolated.** Acme users shouldn't see Globex data.

### Scenario 3: Status-Based Rules

Imagine we want this rule: "Published documents cannot be edited except by admins."

With pure RBAC, there's no way to express this. The `editor` role either has `update` permission or it doesn't—we can't add conditions.

## What RBAC Can Express

```text
✅ "Admins can delete documents"
✅ "Viewers can only read"
✅ "Only admins can manage users"
```

## What RBAC Cannot Express

```text
❌ "Editors can edit their OWN documents"
❌ "Users can only see documents in their organization"
❌ "Published documents cannot be edited"
❌ "Documents can only be edited during business hours"
❌ "Users can edit documents they've been granted access to"
```

## The RBAC Formula

RBAC decisions are simple:

```text
ALLOW if role.permissions.includes(action)
```

There's no consideration of:

- **WHO** owns the resource
- **WHAT** state the resource is in
- **WHERE** the request is coming from
- **WHEN** the request is made
- **WHICH** specific resource is being accessed

## When RBAC Is Enough

RBAC works well for:

- **Simple applications** with few resource types
- **Administrative tools** where admins manage everything
- **Read-heavy applications** where most users just view content
- **Clear organizational hierarchies** (manager > employee > intern)

## When You Need More

You need ABAC when:

- Users should only access **their own** resources
- You have **multi-tenant** requirements
- Access depends on **resource state** (draft, published, archived)
- You need **sharing** or **delegation**
- Rules are **complex or dynamic**

## Our Plan

In the next section, we'll extend our system to support:

1. **Ownership checks** - Editors edit only their documents
2. **Organization scoping** - Users see only their org's data
3. **Field-level permissions** - Control which fields users can see/edit
4. **Status-based rules** - Published docs are read-only for editors

This is Attribute-Based Access Control (ABAC).

## Conceptual Shift

```text
RBAC:  Can user with role X do action Y?
ABAC:  Can user with attributes X do action Y on resource with attributes Z?
```

The key insight is that we're adding **context** to our permission decisions:

- User context: role, id, orgId, department
- Resource context: authorId, orgId, status, createdAt
- Environment context: time, IP address, device

Let's build it!
