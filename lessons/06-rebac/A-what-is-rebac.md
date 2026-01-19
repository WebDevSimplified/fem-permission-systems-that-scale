---
title: "What is ReBAC?"
---

ReBAC (Relationship-Based Access Control) determines permissions based on **relationships between entities** rather than roles or attributes. It's the model behind systems like Google Drive, where sharing a folder automatically grants access to everything inside it.

## The Core Idea

In ReBAC, you define relationships like:

- "Alice is an **owner** of Document X"
- "Bob is a **member** of Team Y"
- "Team Y is **assigned to** Project Z"

Then permissions flow through these relationships:

```text
Can Bob view Project Z?
→ Bob is a member of Team Y
→ Team Y is assigned to Project Z
→ Members of assigned teams can view projects
→ ✅ Yes, Bob can view Project Z
```

## How ReBAC Differs

| Aspect             | RBAC                            | ABAC                              | ReBAC                                    |
| ------------------ | ------------------------------- | --------------------------------- | ---------------------------------------- |
| **Based on**       | User's role                     | Attributes of user/resource       | Relationships between entities           |
| **Question asked** | "What role does the user have?" | "Do attributes match conditions?" | "Is there a path from user to resource?" |
| **Best for**       | Organization hierarchy          | Complex business rules            | Sharing, collaboration, nested resources |

## When ReBAC Shines

ReBAC is ideal when:

- **Sharing is core to your app** - Google Docs, Dropbox, Notion
- **Permissions inherit through hierarchy** - Folder → Subfolder → File
- **Users create their own permission structures** - Workspaces, teams, projects

## Real-World ReBAC

Google Drive's permission model is ReBAC:

```text
Alice shares "Project Folder" with Bob (viewer)
  └── "Design Specs" inherits Bob's access
      └── "Logo.png" inherits Bob's access

Alice shares "Budget.xlsx" with Carol (editor)
  └── Carol can edit, but can't see other files in the folder
```

This would be very complex to model with RBAC or ABAC, but is natural in ReBAC. ReBAC is still very complex to create, though, and is by far the most complex access control model of the three to implement.

## Should You Use ReBAC?

**Yes, if:**

- Your app is fundamentally about sharing and collaboration
- You need hierarchical permission inheritance
- Users need to manage their own access controls

**No, if:**

- Your permissions are role-based (use RBAC)
- Your permissions depend on business rules (use ABAC)
- You don't need sharing or collaboration features

In general, don't bother with ReBAC unless you really need fine-grained, relationship-based access control since it is much harder to implement and basic sharing access can be implemented in ABAC relatively easily.

## Summary

ReBAC is powerful for collaboration-focused applications but adds complexity. For most apps, RBAC or ABAC is simpler and sufficient. Consider ReBAC when your users need to create and manage their own permission structures.

The key insight: **ReBAC models permissions as a graph**, where access is determined by finding a path from the user to the resource through relationships.
