---
title: "Common Permission Mistakes"
---

Before we implement a proper permission system, let's identify the problems in our current codebase. These issues are extremely common in real-world applications—you've likely seen (or written!) code with these same mistakes.

## The Three Cardinal Sins of Permissions

Our demo app currently suffers from three major permission problems:

| Problem                   | Description                                             | Risk Level |
| ------------------------- | ------------------------------------------------------- | ---------- |
| **Scattered checks**      | Permission logic duplicated across files                | High       |
| **Missing server checks** | Permissions only validated on the client                | Critical   |
| **Inconsistent logic**    | Same permission checked differently in different places | High       |

## Problem 1: Scattered Permission Checks

Look at our current codebase. Permission checks are sprinkled throughout:

- Page components (checking if buttons should render)
- Server actions (checking if operations should proceed)
- Route handlers (checking access to pages)

When the same logic exists in multiple places, bugs are inevitable. A developer updates the permission logic in one place but forgets the others. Now you have inconsistent behavior that's incredibly hard to debug.

> ⚠️ **Real-world example**: A common bug is allowing a user to access an "Edit" page but blocking them from actually saving changes. Or worse—blocking the UI button but not the API endpoint, so a savvy user can still perform unauthorized actions via the browser console.

## Problem 2: Missing Server-Side Checks

This is the most dangerous mistake. Our current app has pages that can be accessed by **anyone who knows the URL**, even if the UI doesn't show a link to that page.

For example, if you're logged in as a `viewer` and manually navigate to:

```text
/projects/[projectId]/documents/[documentId]
```

You can access that page (even if you normally wouldn't be able to see that document)

**The golden rule**: Never trust the client. If a permission matters, check it on the server.

## Problem 3: Inconsistent Permission Logic

Our permission checks are complex and hard to read. Consider checking if a user can access a project:

```typescript
// This check appears in multiple places
if (
  user.role !== "admin" &&
  project.department != null &&
  user.department !== project.department
) {
  // deny access
}
```

This logic is:

- **Duplicated** across multiple files
- **Error-prone** (easy to miss a condition)
- **Hard to read** (what does this actually check?)
- **Inconsistent** (some places might check slightly differently)

## Why This Matters

These issues compound over time:

1. **Security vulnerabilities** - Missing checks = unauthorized access
2. **Maintenance nightmare** - Changing permission logic requires updating many files
3. **Testing difficulty** - Hard to verify all permission paths work correctly
4. **Onboarding friction** - New developers don't know where all the checks live

## Our Goals for This Section

By the end of this section, we'll:

1. Add missing server-side permission checks to all pages
2. Identify places where permission logic is inconsistent
3. Recognize the need to centralize this logic (which we'll do properly in later sections)

Let's start by finding and fixing the missing checks in our application.
