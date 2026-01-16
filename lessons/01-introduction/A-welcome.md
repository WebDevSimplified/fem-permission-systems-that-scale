---
title: "Welcome"
description: "Introduction to the Permission Systems That Scale workshop, covering what you'll learn and build throughout the course."
---

# Welcome to Permission Systems That Scale

Welcome! In this workshop, you'll learn how to build robust, maintainable permission systems for real-world web applications. By the end, you'll understand the trade-offs between different authorization approaches and know how to implement them from scratch.

## What You'll Learn

- The fundamental difference between **authentication** and **authorization**
- How to implement **Role-Based Access Control (RBAC)** with clean, type-safe code
- How to build **Attribute-Based Access Control (ABAC)** for complex, dynamic permissions
- When and why to consider **Relationship-Based Access Control (ReBAC)**
- How popular libraries like **CASL** and **Casbin** handle permissions
- How to structure permission logic so it's easy to maintain and extend

## What You'll Build

Throughout this workshop, we'll progressively enhance a document management application. Starting with basic permission checks scattered throughout the codebase, we'll refactor and upgrade the system through multiple iterations:

1. **Basic Permissions** → Inline checks with simple role comparisons
2. **Structured RBAC** → Centralized role-permission mappings
3. **Full ABAC** → Dynamic rules based on user attributes, resource properties, and context
4. **Library Integration** → Migrating to CASL for production-ready permissions

Each upgrade will demonstrate the strengths and limitations of different approaches, helping you choose the right tool for your specific needs.

## Who This Workshop Is For

This workshop is designed for **fullstack web developers** who want to level up their authorization skills. You should be comfortable with:

- **TypeScript** — 95% of the code we write is plain TypeScript
- Basic understanding of databases and SQL concepts
- General web development patterns (APIs, CRUD operations)

> **Note:** While the demo application uses React and Next.js, you don't need prior experience with these frameworks. We'll focus almost entirely on the authorization logic, which is framework-agnostic TypeScript.

## Workshop Format

This is a **hands-on, code-along workshop**. We'll spend most of our time:

- **Live coding** — Building and refactoring permission systems together
- **Discussing trade-offs** — Understanding when to use each approach
- **Exploring edge cases** — Handling real-world permission scenarios

The written materials serve as a reference you can return to after the workshop.

## Let's Get Started

Ready to build permission systems that actually scale? Let's begin by understanding the critical difference between authentication and authorization.
