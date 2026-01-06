---
description: "Implement condition matching logic using simple equality checks."
---

# Condition Matching

Conditions like `{ authorId: user.id }` need to match against the actual resource.

## Live Code: matchesConditions Method

```typescript
// Add to PermissionsImpl class

private matchesConditions(
  conditions: Record<string, unknown> | undefined,
  subject: Record<string, unknown> | null
): boolean {
  // No conditions = matches everything
  if (!conditions) return true;

  // If we have conditions but no subject data, can't match
  if (!subject) return false;

  // Check each condition (all must match = AND logic)
  for (const [key, expectedValue] of Object.entries(conditions)) {
    const actualValue = subject[key];

    if (!this.valuesMatch(actualValue, expectedValue)) {
      return false;
    }
  }

  return true;
}

private valuesMatch(actual: unknown, expected: unknown): boolean {
  // Direct equality
  if (actual === expected) return true;

  // Handle arrays (any match = OR logic)
  if (Array.isArray(expected)) {
    return expected.includes(actual);
  }

  // Handle null/undefined
  if (actual == null && expected == null) return true;

  return false;
}
```

## How It Works

```typescript
// Rule: allow('update', 'Document', { authorId: 5 })

// Check against document
const document = { __typename: "Document", authorId: 5, title: "Hello" }

matchesConditions({ authorId: 5 }, document)
// → true (5 === 5)

const otherDoc = { __typename: "Document", authorId: 10, title: "World" }
matchesConditions({ authorId: 5 }, otherDoc)
// → false (5 !== 10)
```

## Array Conditions

```typescript
// Rule: allow('read', 'Document', { status: ['draft', 'published'] })

const doc1 = { status: "draft" }
matchesConditions({ status: ["draft", "published"] }, doc1)
// → true ('draft' is in the array)

const doc2 = { status: "archived" }
matchesConditions({ status: ["draft", "published"] }, doc2)
// → false ('archived' not in array)
```

This simple equality + array matching covers most real-world needs!
