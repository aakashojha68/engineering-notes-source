# HTTP — Request/Response, Methods, Status Codes, Headers

## Concept
HTTP defines what the bytes TCP delivers actually **mean** — the format of a
request (what am I asking for) and a response (what did you give me back).

## Why it matters
HTTP methods, status codes, and headers are the day-to-day vocabulary of API
work — getting the semantics precisely right (not just "200 means it
worked") is what interviewers probe for, since sloppy status code/method
usage is a real code-review red flag in production APIs.

## Request & Response Structure

```
TCP: "I transport bytes reliably."
HTTP: "I define what those bytes mean."
```

```
Request:                          Response:

Request Line                      Status Line
     ↓                                 ↓
  Headers                           Headers
     ↓                                 ↓
   Body                              Body
```

```
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer xyz

{ "name": "Aakash" }
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{ "id": 1, "name": "Aakash" }
```

## HTTP Methods

| Method | Meaning |
|---|---|
| `GET` | Read |
| `POST` | Create |
| `PUT` | Replace entire resource |
| `PATCH` | Partially update resource |
| `DELETE` | Delete |

**REST semantics** say `PUT` means full replacement and `PATCH` means
partial modification — though some real-world APIs use `PUT` loosely for
partial updates too (worth flagging as an inconsistency you'd correct in review).

## Idempotency

**Idempotent** = repeating the same request leads to the same final resource
state, no matter how many times you send it.

| Method | Idempotent? |
|---|---|
| `GET` | Yes |
| `PUT` | Yes |
| `DELETE` | Yes |
| `POST` | **No** |
| `PATCH` | Depends on implementation |

```
POST /users        sent twice → potentially TWO users created (not idempotent)
PUT /users/1        sent twice → SAME final state both times (idempotent)
```

## Status Codes

```
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client Error
5xx → Server Error
```

| Code | Meaning |
|---|---|
| `200 OK` | Successful request |
| `201 Created` | Resource created |
| `204 No Content` | Successful, no response body |
| `400 Bad Request` | Client sent invalid request/data |
| `401 Unauthorized` | Not authenticated / invalid credentials |
| `403 Forbidden` | Authenticated, but not authorized |
| `404 Not Found` | Requested resource doesn't exist |
| `500 Internal Server Error` | Unexpected server-side failure |

**A precise distinction that trips people up:**

```
GET /users            (zero users exist) → 200 OK
                       (the /users collection itself exists — it's just empty)

GET /users/999         (user 999 doesn't exist) → 404 Not Found
```

**401 vs 403 — the cleanest mental model:**

```
401: "Who are you?"                    → Not authenticated
403: "I know who you are, but no."     → Authenticated, not authorized
```

## Headers

Headers are metadata about the request/response — separate from the actual
content (the body).

```
Analogy:
Package
  ├── Address/weight/etc. = metadata (headers)
  └── Actual item          = body
```

| Request Header | Purpose |
|---|---|
| `Host` | Target host |
| `Content-Type` | Format of the request body |
| `Authorization` | Auth credentials/token |
| `Accept` | Desired response format |
| `User-Agent` | Client information |

| Response Header | Purpose |
|---|---|
| `Content-Type` | Response body format |
| `Content-Length` | Response size |
| `Cache-Control` | Caching behavior |

**Important nuance:** a missing `Content-Type` does **not** automatically
mean a `400` — depending on the framework/server, it may still process
successfully, or return `400 Bad Request` or `415 Unsupported Media Type`.

## Common interview questions
- Difference between `PUT` and `PATCH`?
- Is `POST` idempotent? Why or why not?
- Why does `GET /users` return `200` with an empty array instead of `404`, when there are zero users?
- Explain `401` vs `403` in your own words.
- What's the difference between `Content-Type` and `Accept` headers?

!!! tip "Gotchas / follow-ups"
    - PATCH's idempotency "depends on implementation" is a good nuance to
      surface unprompted — it shows you're not just reciting a table.
    - The empty-collection-still-returns-200 distinction is a favorite
      "gotcha" question precisely because it's counter-intuitive at first glance.
    - Status codes and methods both tie directly into [REST API Design](rest-api-design.md) — expect them together.

## Personal example
_(Add a real case — e.g. a code review catching a `GET` endpoint that had
side effects, or fixing a `PUT` endpoint that wasn't actually idempotent
due to auto-incrementing a counter on every call.)_

## Related
- [REST API Design](rest-api-design.md)
- [Cookies, Sessions & JWT](cookies-and-auth.md)
