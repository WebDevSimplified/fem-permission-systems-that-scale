---
description: "Implement the can() and cannot() methods for checking permissions."
---

# Permission Checking

Now let's implement the core `can()` method.

## Live Code: PermissionsImpl Class

```typescript
// packages/permissions/src/permissions.ts

import { Rule } from "./types"

export class PermissionsImpl<TSubject extends string> {
  constructor(private rules: Rule<TSubject>[]) {}

  can(
    action: string,
    subject: TSubject | Record<string, unknown>,
    field?: string
  ): boolean {
    const subjectType = this.getSubjectType(subject)
    const subjectData = typeof subject === "object" ? subject : null

    // Find matching deny rules first (deny wins)
    const denyRules = this.rules.filter(
      (r) =>
        r.type === "deny" &&
        this.matchesAction(r.action, action) &&
        this.matchesSubject(r.subject, subjectType)
    )

    for (const rule of denyRules) {
      if (this.matchesConditions(rule.conditions, subjectData)) {
        return false
      }
    }

    // Then find matching allow rules
    const allowRules = this.rules.filter(
      (r) =>
        r.type === "allow" &&
        this.matchesAction(r.action, action) &&
        this.matchesSubject(r.subject, subjectType)
    )

    for (const rule of allowRules) {
      if (this.matchesConditions(rule.conditions, subjectData)) {
        // If checking a specific field, verify it's allowed
        if (field && rule.fields && !rule.fields.includes(field)) {
          continue
        }
        return true
      }
    }

    return false
  }

  cannot(
    action: string,
    subject: TSubject | Record<string, unknown>,
    field?: string
  ): boolean {
    return !this.can(action, subject, field)
  }

  authorize(action: string, subject: TSubject | Record<string, unknown>): void {
    if (this.cannot(action, subject)) {
      throw new ForbiddenError(
        `Cannot ${action} ${this.getSubjectType(subject)}`
      )
    }
  }

  private getSubjectType(
    subject: TSubject | Record<string, unknown>
  ): TSubject {
    if (typeof subject === "string") return subject
    if ("__typename" in subject) return subject.__typename as TSubject
    throw new Error("Subject must have __typename or be a string")
  }

  private matchesAction(ruleAction: string, action: string): boolean {
    return ruleAction === action || ruleAction === "manage"
  }

  private matchesSubject(ruleSubject: TSubject, subject: TSubject): boolean {
    return ruleSubject === subject || ruleSubject === ("all" as TSubject)
  }
}

export class ForbiddenError extends Error {
  constructor(message: string) {
    super(message)
    this.name = "ForbiddenError"
  }
}
```

Next, let's implement condition matching.
