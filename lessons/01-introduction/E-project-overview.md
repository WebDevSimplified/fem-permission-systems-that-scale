---
title: "Project Overview"
description: "Understanding the demo application's data model, user roles, and how the UI flows together — the foundation for our permission system work."
---

# Project Overview

Before we start building permission systems, let's understand the application we're working with. It's a simple document management system with users, projects, and documents.

## The Data Model

Our application has three main entities with clear relationships:

![Entity Relationships](/fem-permission-systems-that-scale/images/01-introduction/entity-relationships.svg)

### Users

Users are the people who interact with the application. Each user has:

| Field        | Description                                                      |
| ------------ | ---------------------------------------------------------------- |
| `id`         | Unique identifier                                                |
| `email`      | Login credential                                                 |
| `name`       | Display name                                                     |
| `role`       | One of: `viewer`, `editor`, `author`, `admin`                    |
| `department` | The department they belong to (e.g., "Engineering", "Marketing") |

### Projects

Projects are containers for documents. They're scoped to departments:

| Field         | Description                                                                          |
| ------------- | ------------------------------------------------------------------------------------ |
| `id`          | Unique identifier                                                                    |
| `name`        | Project name                                                                         |
| `description` | What the project is about                                                            |
| `ownerId`     | The user who created the project                                                     |
| `department`  | The department this project belongs to (can be `null` for cross-department projects) |

### Documents

Documents are the core content of the application:

| Field            | Description                                 |
| ---------------- | ------------------------------------------- |
| `id`             | Unique identifier                           |
| `title`          | Document title                              |
| `content`        | The actual document content                 |
| `status`         | One of: `draft`, `published`, `archived`    |
| `isLocked`       | Whether the document is locked from editing |
| `projectId`      | The project this document belongs to        |
| `creatorId`      | The user who created the document           |
| `lastEditedById` | The user who last modified the document     |

## User Roles

The application has four roles with increasing levels of access:

| Role       | Intended Purpose                                |
| ---------- | ----------------------------------------------- |
| **Viewer** | Can view documents but not modify anything      |
| **Editor** | Can edit existing documents                     |
| **Author** | Can create new documents and edit existing ones |
| **Admin**  | Full access to everything                       |

## Department Scoping

Users and projects belong to departments. The intended behavior is:

- Users should primarily see and interact with **their department's projects**
- Some projects have `department: null`, meaning they're **cross-department** and all users can access them

## Desired Permissions

Right now, the application tries to follow the below set of permissions:

### Projects:

| Role   | View | Create | Edit | Delete |
| ------ | ---- | ------ | ---- | ------ |
| Viewer | ✅   | ❌     | ❌   | ❌     |
| Editor | ✅   | ❌     | ❌   | ❌     |
| Author | ✅   | ❌     | ❌   | ❌     |
| Admin  | ✅   | ✅     | ✅   | ✅     |

- All roles other than `admin` can only view projects with a `department` that matches their own department or with `department: null` (cross-department projects)

### Documents:

| Role   | View | Create | Edit | Delete |
| ------ | ---- | ------ | ---- | ------ |
| Viewer | ✅   | ❌     | ❌   | ❌     |
| Editor | ✅   | ❌     | ✅   | ❌     |
| Author | ✅   | ✅     | ✅   | ❌     |
| Admin  | ✅   | ✅     | ✅   | ✅     |

## Next Steps

The next thing we need to work on is fixing the bugs in our permission system and making it more organized to prevent future bugs and make extensions easier in the future.
