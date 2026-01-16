---
title: "Overview of Permission Systems"
description: "A high-level introduction to RBAC, ABAC, and ReBAC — the three main approaches to authorization — and when to use each one."
---

# Overview of Permission Systems

There are three main approaches to implementing authorization, each with different trade-offs between simplicity and flexibility.

![Permission Systems Comparison](/fem-permission-systems-that-scale/images/01-introduction/permission-systems-comparison.svg)

## Role-Based Access Control (RBAC)

**RBAC** assigns permissions based on a user's role. It's the most common and simplest approach.

### How It Works

1. Define **roles** (admin, editor, viewer)
2. Assign **permissions** to each role (can create, can edit, can delete)
3. Assign **roles** to users
4. Check: "Does this user's role have this permission?"

### Example

| Role   | View | Create | Edit | Delete |
| ------ | ---- | ------ | ---- | ------ |
| Viewer | ✅   | ❌     | ❌   | ❌     |
| Editor | ✅   | ❌     | ✅   | ❌     |
| Author | ✅   | ✅     | ✅   | ❌     |
| Admin  | ✅   | ✅     | ✅   | ✅     |

### Best For

- Applications with clear, hierarchical user types
- When permissions don't depend on the specific resource
- Simple permission requirements that change infrequently

### Limitations

- Can't express "editors can only edit documents in their department"
- Role explosion when you need fine-grained control
- Difficult to handle resource-specific permissions

## Attribute-Based Access Control (ABAC)

**ABAC** makes decisions based on attributes of the user, resource, and environment.

### How It Works

1. Define **attributes** on users (department, role, clearance level)
2. Define **attributes** on resources (status, owner, department)
3. Write **policies** that combine attributes with logic
4. Check: "Do the attributes satisfy this policy?"

### Example Policy

```text
ALLOW edit document IF:
  - user.role IN [editor, author, admin]
  - AND document.status != "archived"
  - AND (document.department == user.department OR user.role == "admin")
  - AND document.isLocked == false
```

### Best For

- Complex, dynamic permission requirements
- Multi-tenant applications
- When permissions depend on resource properties
- Fine-grained, field-level access control

### Limitations

- More complex to implement and understand
- Policies can become hard to audit
- Performance considerations with many attribute checks

## Relationship-Based Access Control (ReBAC)

**ReBAC** determines access based on the relationships between users and resources.

### How It Works

1. Define **relationship types** (owner, member, viewer)
2. Create **relationships** between users and resources
3. Define **permission rules** based on relationships
4. Check: "Does this user have a relationship that grants this permission?"

### Example

```text
User "alice" --[owner]--> Project "docs"
Project "docs" --[contains]--> Document "readme"

Rule: owner of project can edit documents in that project
Result: Alice can edit "readme"
```

### Best For

- Social applications (who can see whose posts?)
- Collaborative tools (shared folders, team workspaces)
- When permissions flow through object hierarchies
- Google Drive-style sharing models

### Limitations

- Requires a relationship graph (often a separate database)
- Can be overkill for simpler permission models
- More infrastructure complexity

## Policy Engines

Every authorization needs a policy engine that takes in all the parameters of the access request (user, resource, action, context) and evaluates them against the defined policies to make an allow/deny decision. These engines usually take one of two forms.

### Domain Specific Language (DSL)

A domain-specific language is a specialized language designed to express authorization policies. It allows you to write rules in a concise, human-readable format that the policy engine can interpret.

#### Example

```text
allow(user, action, resource) if
  user.role == "admin" or
  (user.role == "editor" and resource.department == user.department)
```

### In-Language Engine

An in-language engine is a policy engine implemented directly within the application's programming language. Instead of using a separate DSL, policies are expressed using the language's native constructs, such as functions, classes, and conditionals.

#### Example

```javascript
function canEdit(user, resource) {
  return (
    user.role === "admin" ||
    (user.role === "editor" && resource.department === user.department)
  )
}
```

## Key Takeaways

- **RBAC** is simple but limited — great for basic apps with clear roles
- **ABAC** is flexible but complex — needed for dynamic, attribute-based rules
- **ReBAC** models relationships — ideal for collaborative/social features
- Most real apps end up with a **hybrid** approach

Next, let's get the workshop project set up on your machine.
