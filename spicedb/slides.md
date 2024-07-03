---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: SpiceDB
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# SpiceDB

<div class="pt-12">
  <span @click="next" class="px-2 p-1 rounded cursor-pointer hover:bg-white hover:bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

---
layout: intro
---

# Agenda

- Identification, Authentication, **Authorization**
- Kind of **Authentication**
- Zanzibar
- SpiceDB as a solution

---

# Identification, Authentication, Authorization

- **Identification** - Who are you? (@batazor)
- **Authentication** - How do you prove it? (password, token, etc)
- **Authorization** - What are you allowed to do? (RBAC, ABAC, etc)

<br />

### Flow

Does `<subject>` have `<permission>` to `<object>`?

**Example:**

> Does `<@batazor>` have permission `<add star>` to `<@spicedb>`?

---

# Authentication + Identification

- Basic Auth
- JWT (go-jwt, etc...)
- OAuth2 (go-oauth2, passportjs, etc...)
- OpenID Connect (go-oidc, etc...)

---

# Authorization

- https://casbin.org/
  - RBAC, ABAC, etc...
  - Go, NodeJS, Python, etc...
- **usually it's handmade**

---
layout: intro
---

# Kind of Authentication

- RBAC
- ABAC
- ReBAC
- ACL
- other standards...

---

# Kind of Authentication: RBAC (Role-Based Access Control)

![rbac](https://miro.medium.com/v2/resize:fit:601/1*ub3g0nUC6NCkYPHoNB6rFw.png)

---

# Kind of Authentication: RBAC (Role-Based Access Control)

+ Pros
  - Easy to implement
  - Easy to understand
- Cons
  - Not flexible
  - Not scalable
  - Not dynamic

---

# Kind of Authentication: ABAC (Attribute-Based Access Control)

![abac](https://cloudradius.com/wp-content/uploads/2022/09/ABAC-Image-1-2.png)

---

# Kind of Authentication: ABAC (Attribute-Based Access Control)

+ Pros
  - Flexible
  - Scalable
  - Dynamic
- Cons
  - Complex
  - Hard to implement
  - Hard to understand

---

# Kind of Authentication: ReBAC (Relationship-Based Access Control)

![rebac](https://media.graphassets.com/gTrUj0fZRV6RgqdKvXYv)

---
layout: two-cols-header
---

# Kind of Authentication: ReBAC (Relationship-Based Access Control)

::left::

+ Pros
  - Flexible
  - Scalable
  - Dynamic
- Cons
  - Complex
  - Hard to implement
  - Medium to understand

::right::

### Entities:

- Subject
- Action
- Object
- Relationship

---
layout: intro
---

# Zanzibar

- [Zanzibar: Google’s Consistent, Global Authorization System](https://research.google/pubs/zanzibar-googles-consistent-global-authorization-system/)
- [Authzed: zanzibar](https://authzed.com/zanzibar)
- [Understanding Google Zanzibar: A Comprehensive Overview](https://authzed.com/blog/what-is-google-zanzibar)
- [Jake Moshenko on Zanzibar: Google’s Consistent, Global Authorization System](https://youtu.be/1nbSbe3kw2U?si=KpHYg7Ui7AOdcsGi)

---

# Zanzibar: goals

- Correctness (step-by-step)
- Flexibility (policy)
- Low latency (3mc; 99% -> 20mc)
- High availability
- Scalability (20m+ rps)

---

# Zanzibar: architecture

![architecture](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rKxFiyEIW240TDoBpPscYg.png)

---

# Zanzibar: features

- Relation Tuples (Subject, Action, Object)
- Namespace Configuration (Domain)
- Good API (Check, Write, Read, Watch)

---
layout: image-right
---

# Implementation of Zanzibar

- [SpiceDB](https://spicedb.com/)
- [Ory Keto](https://www.ory.sh/keto/)
- and more...

---

# SpiceDB

- Open Source
- 4.7k stars
- UI (graph visualization + editor + test cases + save to file)
- SpiceDB Operator for Kubernetes
- Extentention for VSCode (Language Server)
- Wildcard policy

### References

- [ABAC on SpiceDB: Enabling Netflix’s Complex Identity Types](https://netflixtechblog.com/abac-on-spicedb-enabling-netflixs-complex-identity-types-c118f374fa89)

---

# SpiceDB: Schema

```graphql
// user represents a user that can be granted role(s)
definition user {}

// document represents a document protected by Authzed.
definition document {}
```

---

# SpiceDB: Schema

```graphql
// user represents a user that can be granted role(s)
definition user {}

// document represents a document protected by Authzed.
definition document {
    // writer indicates that the user is a writer on the document.
    relation writer: user

    // reader indicates that the user is a reader on the document.
    relation reader: user
}
```

---

# SpiceDB: Schema

```graphql
// user represents a user that can be granted role(s)
definition user {}

// document represents a document protected by Authzed.
definition document {
    // writer indicates that the user is a writer on the document.
    relation writer: user

    // reader indicates that the user is a reader on the document.
    relation reader: user

    // edit indicates that the user has permission to edit the document.
    permission edit = writer

    // view indicates that the user has permission to view the document, if they
    // are a `reader` *or* have `edit` permission.
    permission view = reader + edit
}
```

---

# SpiceDB: Tuple

### Flow

Does `<subject>` have `<permission>` to `<object>`?

### Format

> `<resource>:<id>#<relation>@<subject>:<id>`

### Example

```graphql
document:firstdoc#writer@user:tom
document:firstdoc#reader@user:fred
document:seconddoc#reader@user:tom
```

---

# SpiceDB: In real life

1. Load schema to SpiceDB by API
2. Load tuples to SpiceDB by API
3. Check permission by API
4. PROFIT

---

# Golang: example

```go 
relationship := &permission.WriteRelationshipsRequest{
    Updates: []*permission.RelationshipUpdate{{
        Operation: permission.RelationshipUpdate_OPERATION_CREATE,
        Relationship: &permission.Relationship{
            Resource: &permission.ObjectReference{
                ObjectType: "document",
                ObjectId:   document.GetId(),
            },
            Relation: "writer",
            Subject: &permission.SubjectReference{
                Object: &permission.ObjectReference{
                    ObjectType: "user",
                    ObjectId:   user.GetId(),
                },
            },
        },
    }},
}

authClient.WriteRelationships(ctx, relationship)
authClient.DeleteRelationships(ctx, relationship)
authClient.CheckPermission(ctx, relationship)
authClient.LookupResources(ctx, relationship)
```

---
layout: intro
---

# SpiceDB: Playgrounds

<div class="pt-12">
  <span @click="next" class="px-2 p-1 rounded cursor-pointer hover:bg-white hover:bg-opacity-10">
    <a href="https://play.authzed.com/schema" target="_blank">Open Playground</a>
  </span>
</div>

---
layout: center
---

# Learn More

- [SpiceDB Docs](https://authzed.com/docs/spicedb/getting-started/discovering-spicedb)
- [ABAC on SpiceDB: Enabling Netflix’s Complex Identity Types](https://netflixtechblog.com/abac-on-spicedb-enabling-netflixs-complex-identity-types-c118f374fa89)
- [Почему авторизация сложно и причем здесь Занзибар? -Максим Горозий, Тинькофф](https://youtu.be/Tr5H8iG0FzI?si=WjvRZ554IpmPWEDd)
