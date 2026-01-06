---
description: "An overview of the major permission system approaches: RBAC, ABAC, and ReBAC, with guidance on when to use each."
---

# Permission Systems Overview

There are several approaches to implementing permissions. Each has trade-offs between simplicity and flexibility.

## The Three Main Approaches

### RBAC: Role-Based Access Control

**Concept:** Users are assigned roles, roles have permissions.

```text
User → Role → Permissions

Example:
  Alice → Editor → [read, create, update]
  Bob → Viewer → [read]
  Carol → Admin → [read, create, update, delete, manage]
```

**Pros:**

- Simple to understand and implement
- Easy to audit ("who has admin access?")
- Works well for organizations with clear hierarchies

**Cons:**

- Coarse-grained (editor can edit ALL documents)
- Role explosion in complex systems
- Can't express "only your own resources"

**Best for:** Simple applications, clear organizational roles, when you need easy auditing.

---

### ABAC: Attribute-Based Access Control

**Concept:** Decisions based on attributes of the user, resource, action, and environment.

```text
User Attributes + Resource Attributes + Context → Decision

Example:
  Can update if:
    - user.role == "editor" AND
    - document.authorId == user.id AND
    - document.status != "published"
```

**Pros:**

- Fine-grained control
- Can express complex business rules
- Handles ownership, time-based rules, location, etc.

**Cons:**

- More complex to implement
- Harder to audit and reason about
- Policy can become sprawling

**Best for:** Multi-tenant apps, ownership-based access, complex business rules.

---

### ReBAC: Relationship-Based Access Control

**Concept:** Access based on relationships between entities in a graph.

```text
User --[member_of]--> Organization --[owns]--> Document
User --[shared_with]--> Document

Can view Document if:
  - User is author of Document, OR
  - User is member of Organization that owns Document, OR
  - Document is shared with User
```

**Pros:**

- Natural for social/collaborative apps
- Handles sharing and nested access well
- Scales to complex hierarchies

**Cons:**

- Most complex to implement
- Requires graph traversal
- Can be slow without optimization

**Best for:** Collaborative apps, document sharing, social networks, complex organizational hierarchies.

---

## Comparison Table

| Aspect              | RBAC  | ABAC   | ReBAC |
| ------------------- | ----- | ------ | ----- |
| **Complexity**      | Low   | Medium | High  |
| **Flexibility**     | Low   | High   | High  |
| **"Own resources"** | ❌    | ✅     | ✅    |
| **Sharing**         | ❌    | ⚠️     | ✅    |
| **Audit**           | Easy  | Medium | Hard  |
| **Implementation**  | Hours | Days   | Weeks |

## What We'll Build

In this workshop, we'll implement RBAC first, then evolve it to ABAC with field-level permissions. ReBAC is covered as an optional section.

The key insight is that these aren't mutually exclusive. Most real applications use a **combination**:

- RBAC for broad access categories
- ABAC for fine-grained rules and ownership
- ReBAC for sharing features (if needed)

## Common Terminology

Before we continue, let's define some terms we'll use throughout:

| Term          | Definition                                                  |
| ------------- | ----------------------------------------------------------- |
| **Subject**   | The entity trying to perform an action (usually a user)     |
| **Resource**  | The thing being accessed (document, comment, user profile)  |
| **Action**    | What the subject wants to do (read, create, update, delete) |
| **Policy**    | Rules that determine if an action is allowed                |
| **Condition** | Additional constraints on a rule (ownership, status, time)  |

Now let's look at the application we'll be adding permissions to.
