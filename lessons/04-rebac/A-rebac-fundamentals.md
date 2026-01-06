---
description: "Understand relationship-based access control and how it models permissions as a graph of relationships."
---

# ReBAC Fundamentals

ReBAC models permissions as relationships between entities in a graph. This is ideal for sharing and collaborative features.

## The Core Concept

Instead of "user has role" or "user has attribute", ReBAC asks: "what is the **relationship** between user and resource?"

```text
User --[relationship]--> Resource

Examples:
  Alice --[owner]--> Document1
  Bob --[viewer]--> Document1
  Alice --[member]--> Acme Corp --[owns]--> Document1
```

## Relationship Tuples

ReBAC systems store relationships as tuples:

```text
(subject, relation, object)

Examples:
  (user:alice, owner, document:1)
  (user:bob, viewer, document:1)
  (user:alice, member, org:acme)
  (org:acme, parent, document:1)
```

## Permission Resolution

"Can Alice view Document1?" becomes a graph traversal:

```text
1. Is Alice directly a viewer of Document1? No
2. Is Alice an owner of Document1? Yes! → Owners can view
   → ALLOW

Or:
1. Is Bob directly a viewer of Document1? Yes! → ALLOW
```

## Nested Relationships

The power of ReBAC is handling nested access:

```text
Can Alice edit Document1?

1. Direct: Alice --[editor]--> Document1? No
2. Via org: Alice --[member]--> AcmeCorp --[owns]--> Document1?
   - Alice is member of AcmeCorp? Yes
   - AcmeCorp owns Document1? Yes
   - Members can edit owned docs? Yes
   → ALLOW
```

## When to Use ReBAC

- **Document sharing** (Google Docs style)
- **Team/organization hierarchies**
- **Social features** (followers, friends)
- **Complex delegation**

## Trade-offs

| Pros                | Cons                        |
| ------------------- | --------------------------- |
| Natural for sharing | More complex to implement   |
| Handles hierarchies | Graph traversal can be slow |
| Very flexible       | Harder to audit             |

Let's add basic sharing to our app.
