# REST API Design

## Concept
REST (Representational State Transfer) is a set of architectural guidelines
for designing HTTP APIs around a simple core mindset: **URLs represent
resources (nouns), HTTP methods represent actions (verbs)**.

## Why it matters
Bad REST design (action-based URLs, misused methods, stateful assumptions)
is one of the most common things flagged in real API code reviews — and
"design a REST API for X" is a frequent live-coding/whiteboard exercise at
this experience level.

## Core Mindset

```
URL           = Resource
HTTP Method   = Action
```

```
Bad (action-based URLs):        Better (resource-based):

/createUser                     POST   /users
/deleteOrder                    DELETE /orders/1
/updateUser                     PATCH  /users/1
```

Resources should be **nouns**: `/users`, `/products`, `/orders`, `/books` —
never verbs.

## CRUD Mapping

```
GET    /users        → Get collection
GET    /users/10      → Get one user
POST   /users        → Create user
PUT    /users/10      → Replace user 10
PATCH  /users/10      → Partially update user 10
DELETE /users/10      → Delete user 10
```

## Nested Resources

```
GET /users/25/orders

users
  │
  ▼
user 25
  │
  ▼
orders
```

This URL shape directly expresses the relationship — "orders belonging to
user 25" — without needing a separate query parameter or a different endpoint shape.

## Path Parameters vs Query Parameters

```
Path parameter → WHICH resource?
Query parameter → HOW should I filter/search/sort the collection?
```

```
/users/10                     → 10 is a path parameter (identifies ONE resource)

/users?page=2                 → query parameter (pagination)
/orders?status=completed      → query parameter (filtering)
/products?category=mobile     → query parameter (filtering)
```

## REST Statelessness

**Every request must contain all information the server needs to process
it.** The server should not rely on remembering anything from a client's
previous requests.

```
Stateful (session-based):

Request → Session ID → Server session storage → User identity
(server must REMEMBER something between requests)

Stateless (JWT-based):

Request → JWT → Verify token → Process request
(everything needed is IN this single request)
```

**Why statelessness matters for scaling:**

```
Load Balancer
     │
     ├──► Server A
     ├──► Server B
     └──► Server C
```

Any server can independently verify a JWT — no server needs to have seen
this specific client before, since there's no local session to look up.
This is what makes stateless architecture trivially horizontally scalable,
compared to session-based auth, which typically needs sticky sessions or
shared session storage.

## Common interview questions
- Design a REST API for a simple e-commerce order system (users, orders, products).
- What makes an API "RESTful" versus just "an API that uses HTTP"?
- Why does REST favor nouns over verbs in URLs?
- Explain REST statelessness and why it matters for horizontal scaling.
- When would you use a path parameter versus a query parameter?

!!! tip "Gotchas / follow-ups"
    - "RESTful" is often used loosely in the industry for any HTTP JSON API —
      being able to articulate what makes something *actually* RESTful
      (resource-based URLs, proper method semantics, statelessness) shows real understanding.
    - Nested resources shouldn't go more than 2–3 levels deep in practice
      (`/users/25/orders/10/items` is already getting unwieldy) — a good
      follow-up point to raise if asked to design something complex.
    - This topic connects directly to [HTTP Basics](http-basics.md) (methods,
      idempotency, status codes) and [Cookies, Sessions & JWT](cookies-and-auth.md)
      (why stateless auth matters here specifically).

## Personal example
_(Add a real case — e.g. redesigning a legacy action-based API
`/getUserOrders` into proper resource-based routes during a Spring Boot
refactor, or a config-driven module you built that mapped cleanly to REST resources.)_

## Related
- [HTTP Basics](http-basics.md)
- [Cookies, Sessions & JWT](cookies-and-auth.md)
- [CORS](cors.md)
