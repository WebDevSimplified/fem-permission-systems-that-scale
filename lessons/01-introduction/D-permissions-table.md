# Initial Permissions Table

This table outlines the baseline permissions for our system. These permissions will evolve as we explore different permission models throughout the course.

## Roles and Resources

**Roles:** Viewer, Author, Editor, Admin  
**Resources:** Documents, Projects

## Permissions Matrix

| Role       | Documents                    | Projects                           |
| ---------- | ---------------------------- | ---------------------------------- |
| **Admin**  | Create, Read, Update, Delete | Create, Read, Update, Delete       |
| **Author** | Create, Read, Update         | Read (filtered by user.department) |
| **Editor** | Read, Update                 | Read (filtered by user.department) |
| **Viewer** | Read                         | Read (filtered by user.department) |

## Permission Details

### Admin

- **Documents:** Full CRUD access to all documents
- **Projects:** Full CRUD access to all projects

### Author

- **Documents:** Can create, read, and update all documents
- **Projects:** Can only read projects where `project.department === user.department`

### Editor

- **Documents:** Can read and update all documents
- **Projects:** Can only read projects where `project.department === user.department`

### Viewer

- **Documents:** Can read all documents
- **Projects:** Can only read projects where `project.department === user.department`

## Notes

- Only Admins can delete any resources
- Only Admins and Authors can create documents
- Projects have department-based access control for non-admin users
- Documents are globally accessible (at minimum for reading) to all authenticated users

## Problems with Current Implementation

The starter application has several security vulnerabilities that we'll address throughout this course:

### 1. Missing Backend Authorization Checks

The application doesn't verify user access to resources on the backend:

- **Project pages:** View and edit pages don't check if the user has permission to access the project
- **Document pages:** View, new, and edit pages don't validate user permissions before serving data

**Risk:** Users can access resources by directly navigating to URLs, bypassing any frontend restrictions.

### 2. Missing Frontend Permission Checks

The UI doesn't conditionally render based on user permissions:

- **Document pages:** View, edit, and new pages show the same UI to all users regardless of their role
- **Project pages:** New and edit pages are visible to users who shouldn't have access to them

**Risk:** Users see forms and actions they can't (or shouldn't be able to) perform, leading to poor UX and potential confusion.

### 3. Incomplete Role Validation in DAL

The `createDocument` data access layer function has a bug:

- Missing the "viewer" role check in permission validation
- This could allow viewers to create documents if the frontend check is bypassed

**Risk:** Authorization logic gaps can be exploited to perform unauthorized actions.
