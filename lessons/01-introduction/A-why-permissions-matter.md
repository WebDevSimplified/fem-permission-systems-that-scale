---
description: "Learn why authorization is critical for web application security and how broken access control became the #1 vulnerability in OWASP's Top 10."
---

# Why Permissions Matter

Welcome to this workshop on permission systems! Before we dive into code, let's understand why this topic is so important.

## Authorization vs Authentication

These two terms are often confused, but they're fundamentally different:

- **Authentication** answers: "Who are you?"
- **Authorization** answers: "What are you allowed to do?"

Authentication verifies identity (login, passwords, OAuth). Authorization determines access rights. You can have perfect authentication and still have broken authorization.

## Broken Access Control: OWASP #1

In 2021, **Broken Access Control** moved to the #1 position in the OWASP Top 10 security risks, up from #5 in 2017. This means it's the most common and impactful vulnerability in modern web applications.

### Real-World Examples

**Example 1: The Parler Data Breach (2021)**
When Parler was taken offline, researchers discovered that posts were numbered sequentially (`/post/1`, `/post/2`, etc.) with no authorization checks. Anyone could enumerate and download every post, including deleted ones and private messages.

**Example 2: The First American Financial Breach (2019)**
885 million sensitive documents (Social Security numbers, bank statements, mortgage records) were accessible by simply changing a document ID in the URL. No authorization check existed.

**Example 3: The Facebook View As Bug (2018)**
A flaw in the "View As" feature allowed attackers to steal access tokens for 50 million accounts. The permission boundary between viewing _as_ someone and _being_ someone was broken.

## The Cost of Getting It Wrong

- **Data breaches** expose sensitive user information
- **Regulatory fines** (GDPR, HIPAA, PCI-DSS) can be millions of dollars
- **Reputation damage** destroys user trust
- **Legal liability** for negligence

## What We'll Learn

In this workshop, we'll build a real permission system from scratch. You'll learn:

1. **RBAC** (Role-Based Access Control) - The foundation
2. **ABAC** (Attribute-Based Access Control) - Adding context and conditions
3. **Field-level permissions** - Controlling which data users can see and edit
4. **Existing libraries** - CASL and Casbin
5. **Building your own** - A custom permission engine

By the end, you'll have the knowledge to implement robust authorization in any application.

## The Golden Rule

> **Never trust the client. Always verify on the server.**

We'll see this rule in action throughout the workshop, including demonstrations of what happens when you break it.

Let's get started!
