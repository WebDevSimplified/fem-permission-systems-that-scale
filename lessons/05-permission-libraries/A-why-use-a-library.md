---
title: "Why Use a Permission Library?"
---

We've built our own ABAC system from scratch, which is great for understanding how permissions work. But in production, you might want to use a battle-tested library instead.

## When to Use a Library

Building your own permission system makes sense when:

- Learning how permissions work (like this course!)
- You need maximum control over your permission logic
- Your permissions aren't too complex

Using a library makes sense when:

- You want well-tested, production-ready code
- You need advanced features
- You want to follow established patterns
- Your team needs documentation and community support

## Two Types of Permission Libraries

Permission libraries generally fall into two main categories based on how you define policies:

### 1. In-Code Libraries

These libraries define permissions using the same programming language as your application:

```typescript
// Permissions defined in JavaScript/TypeScript
const ability = defineAbility((can, cannot) => {
  can("read", "Document", { status: "published" })
  can("update", "Document", { creatorId: user.id })
  cannot("delete", "Document", { isLocked: true })
})
```

**Pros:**

- Full IDE support (autocomplete, type checking)
- Easy to debug (same language, same tools)
- No context switching between languages
- Can use application logic directly

**Cons:**

- Policies are embedded in application code
- Harder to externalize or share policies
- Changes require redeploying the application (unless using a dynamic configuration system)

**Popular in-code libraries:**

- **CASL** (JavaScript/TypeScript)
- **Pundit** (Ruby)
- **Cancan** (Ruby)

### 2. DSL Libraries (Domain-Specific Language)

These libraries define permissions using a dedicated policy language:

```ini
# model.conf - Defines the permission structure
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

```csv
# policy.csv - The actual rules
p, admin, document, read
p, admin, document, update
p, admin, document, delete
p, editor, document, read
p, editor, document, update
```

**Pros:**

- Policies are separate from application code
- Can update policies without redeploying
- Same policies work across different services
- Easier to audit (policies in one place)

**Cons:**

- Must learn a new language
- Limited IDE support (usually)
- Debugging across language boundary is harder
- Extra infrastructure to manage policy files

**Popular DSL libraries:**

- **Casbin** - Supports multiple languages
- **Open Policy Agent (OPA)** - Uses Rego language
- **Cedar** - AWS's policy language

## Our Example Libraries

You may have already noticed our ABAC system is an in code system and is actually heavily inspired by CASL which we will be implementing in this course.

### CASL (In-Code)

CASL is a popular JavaScript/TypeScript library for handling permissions:

```typescript
import { AbilityBuilder, createMongoAbility } from "@casl/ability"

const { can, build } = new AbilityBuilder(createMongoAbility)

can("read", "Document", { status: "published" })
can("update", "Document", { creatorId: user.id, isLocked: false })

const ability = build()

// Check permissions
ability.can("read", subject("Document", document))
ability.can("update", subject("Document", document))
```

### Casbin (DSL)

Casbin uses a model/policy separation where you define:

1. **Model file** - The structure of your permissions
2. **Policy file** - The actual rules

```ini
# model.conf - Defines the structure
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

```csv
# policy.csv - The actual rules
p, admin, document, read
p, admin, document, create
p, admin, document, update
p, admin, document, delete
p, editor, document, read
p, editor, document, update
```

Casbin's approach is much harder to parse until you learn the language around it, but it is more flexible since you can use it across multiple languages and services.

## Which Should You Choose?

| Factor             | In-Code (CASL)          | DSL (Casbin)           |
| ------------------ | ----------------------- | ---------------------- |
| **Learning curve** | Easy if you know JS     | Must learn new syntax  |
| **Type safety**    | Full TypeScript support | Limited                |
| **Policy updates** | May require a redeploy  | Can update dynamically |
| **Multi-service**  | Harder to share         | Built for this         |
| **Debugging**      | Same tools              | Harder                 |

**Choose in-code (CASL) when:**

- Your app is monolithic or has few services that use other languages
- You want maximum type safety
- Your team is JavaScript/TypeScript focused

**Choose DSL (Casbin) when:**

- You have multiple services sharing policies across different languages
- You need to update policies without deploying
- You have a security team managing policies

## What's Next

In the next lesson, we'll dive deeper into how to implement CASL in our existing application.
