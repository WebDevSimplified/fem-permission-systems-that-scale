---
description: "Overview of Casbin's DSL-based approach to authorization with external policy files."
---

# Casbin Overview

Casbin takes a different approach: **DSL-based configuration** with external policy files. This separates the authorization model from your code.

## Philosophy

- **Model + Policy separation**: Define the model once, policies are data
- **Multi-language**: Same model works in Go, Java, Python, Node, etc.
- **Multiple models**: ACL, RBAC, ABAC all supported
- **Policy storage**: File, database, or API

## The PERM Model

Casbin uses four components:

```text
P - Policy: The rules (who can do what)
E - Effect: How to combine rule results
R - Request: What's being asked (sub, obj, act)
M - Matchers: How to match requests to policies
```

## Model File Example

```ini
# model.conf

[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

## Policy File Example

```csv
# policy.csv
p, alice, document:1, read
p, alice, document:1, update
p, bob, document:1, read
p, admin, document:*, *
```

## Basic Usage

```typescript
import { newEnforcer } from "casbin"

const enforcer = await newEnforcer("model.conf", "policy.csv")

const allowed = await enforcer.enforce("alice", "document:1", "read")
// true
```

## When to Use Casbin

- **Multi-language systems**: Same policies across microservices
- **Non-dev policy management**: Policies as data, not code
- **Complex RBAC models**: Role hierarchies, domains

## CASL vs Casbin

| Aspect         | CASL      | Casbin    |
| -------------- | --------- | --------- |
| Config         | Code      | Files/DSL |
| TypeScript     | Excellent | Basic     |
| Learning curve | Lower     | Higher    |
| Flexibility    | High      | Very high |
| Policy storage | In-memory | Adapters  |

For most JavaScript apps, CASL is simpler. Casbin shines in polyglot environments.
