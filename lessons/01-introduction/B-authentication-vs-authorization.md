---
title: "Authentication vs Authorization"
description: "Understanding the critical difference between authentication (who are you?) and authorization (what can you do?), and why authorization is often the harder problem."
---

# Authentication vs Authorization

Before diving into permission systems, we need to clearly understand two concepts that are often confused: **authentication** and **authorization**.

## The Key Distinction

| Concept            | Question It Answers | Example                             |
| ------------------ | ------------------- | ----------------------------------- |
| **Authentication** | "Who are you?"      | Logging in with email/password      |
| **Authorization**  | "What can you do?"  | Can this user delete this document? |

Think of it like entering a secure building:

- **Authentication** is showing your ID badge at the entrance — proving you are who you claim to be
- **Authorization** is the access levels on your badge — determining which floors and rooms you can enter

## Authentication Is (Mostly) Solved

Authentication has well-established patterns and battle-tested solutions:

- **OAuth/OIDC** providers (Google, GitHub, etc.)
- **Managed services** (Auth0, Clerk, Firebase Auth)
- **Session management** libraries
- **JWT** standards

For most applications, you can integrate an authentication provider and move on. The hard problems — password hashing, session security, MFA — are handled for you.

## Authorization Is Where Complexity Lives

Authorization is a fundamentally different challenge because:

### 1. It's Deeply Tied to Business Logic

Every application has unique rules:

- "Editors can modify documents, but only in their department"
- "Authors can publish their own drafts, but need approval for others"
- "Admins can do everything, except delete locked documents"

These rules can't be outsourced to a generic provider — they're specific to _your_ application.

### 2. It Touches Every Part of Your Codebase

Permission checks appear everywhere:

- **API routes** — Can this user call this endpoint?
- **Database queries** — Which records should be returned?
- **UI components** — Should this button be visible?
- **Business logic** — Is this state transition allowed?

### 3. It Evolves Constantly

Business requirements change:

- "We need to add a new role for external contractors"
- "Documents should be locked after 30 days"
- "Department heads can now approve cross-department access"

A rigid permission system becomes a bottleneck for feature development.

## The Cost of Getting It Wrong

Poor authorization leads to real problems:

| Issue                | Consequence                                                  |
| -------------------- | ------------------------------------------------------------ |
| **Over-permissive**  | Data breaches, unauthorized actions, compliance violations   |
| **Under-permissive** | Frustrated users, support tickets, lost productivity         |
| **Inconsistent**     | Security holes where checks are missing, confusing UX        |
| **Unmaintainable**   | New features require touching dozens of files, bugs multiply |

## This Workshop's Focus

This workshop is **100% about authorization**. We assume you already have authentication handled — users can log in and you know who they are.

Our challenge is: given a known user, how do we efficiently and maintainably determine what they're allowed to do?

## Key Takeaways

- **Authentication** verifies identity; **authorization** controls access
- Authentication is largely a solved problem with good tooling
- Authorization is application-specific and requires careful design
- Poor authorization creates both security risks and maintenance nightmares

Next, let's preview the different authorization systems we'll explore in this workshop.
