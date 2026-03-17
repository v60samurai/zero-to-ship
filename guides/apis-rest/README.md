# APIs and REST: Zero to Ship

> One program asking another program to do something over the internet, like ordering food through a waiter instead of walking into the kitchen yourself.

## Why Should You Care?

Every app you've ever used talks to APIs. When you open Instagram, the app on your phone makes dozens of API calls: fetch the feed, load stories, check notifications, pull comments. When you sign in with Google on a random website, that's an API call. When Cursor autocompletes your code, it's calling an AI API. When your Next.js app loads data on the server, it's hitting an API. APIs are the plumbing behind everything.

If you build with AI tools, you're already generating API code constantly. Cursor writes fetch calls, Claude Code builds API routes, v0 generates forms that submit data somewhere. But when something breaks (and it will), you need to understand what's happening. Why did the request fail? What does a 403 mean? Why is the data coming back wrong? Why does the API work in Postman but not in your app? You can't debug what you don't understand.

REST is the most common way APIs are designed. It's not the only way (there's GraphQL, gRPC, WebSockets), but it's what you'll encounter 90% of the time. Understanding REST means understanding the language that most backend services speak. It's the foundation for everything else: authentication, webhooks, integrations, microservices, and yes, the LLM APIs you're probably already using.

## The 30-Second Version

An API is a way for two programs to talk to each other. One program sends a request ("give me this user's data" or "create a new order"), and the other sends back a response (the data, or a confirmation, or an error). REST is a set of conventions for designing these APIs: use URLs to identify things (resources), use HTTP methods to describe what you want to do (GET to read, POST to create, PUT to update, DELETE to remove), and use status codes to communicate what happened (200 means success, 404 means not found, 500 means something broke on the server).

## The Real Explanation (ELI5)

Imagine you're at a restaurant. You don't walk into the kitchen and start cooking. You sit at a table, look at the menu, and tell the waiter what you want. The waiter takes your order to the kitchen, the kitchen prepares it, and the waiter brings back your food (or comes back and says "sorry, we're out of the paneer tikka").

In this analogy:

- **You** are the client (a frontend app, a mobile app, or another server).
- **The waiter** is the API. It's the interface between you and the kitchen.
- **The kitchen** is the server (the backend, the database, the business logic).
- **The menu** is the API documentation. It tells you what you can order and how to order it.
- **Your order** is the request. "I'd like the paneer tikka" is like `GET /menu/paneer-tikka`.
- **The food** (or the "sorry, we're out") is the response.

REST adds some conventions to this restaurant. Instead of just yelling your order, you follow a specific protocol:

- **GET** = "Show me what you have." (reading the menu, checking your order status)
- **POST** = "I'd like to place a new order." (creating something)
- **PUT** = "Change my order to something else." (replacing/updating something)
- **DELETE** = "Cancel my order." (removing something)

And the waiter always comes back with a status:

- **200** = "Here's your food." (success)
- **201** = "Order placed successfully." (created)
- **400** = "That's not a real menu item." (bad request, your fault)
- **404** = "We don't have that." (not found)
- **500** = "The kitchen is on fire." (server error, their fault)

Now replace the restaurant with the internet, the waiter with HTTP, and the menu with API documentation. That's REST.

The analogy breaks at one important point: in a restaurant, the waiter remembers your table, your previous orders, and your preferences. REST APIs don't. Each request is completely independent. The server doesn't remember your last request. If you need context (like being logged in), you have to send proof with every single request (usually a token in the header). This is called "statelessness," and it's a feature, not a bug, because it makes the system simpler and more scalable.

## How It Actually Works

### HTTP Basics

Every API request is an HTTP message. HTTP (HyperText Transfer Protocol) is the language that web browsers, apps, and servers use to talk to each other. An HTTP request has four parts:

```
┌─────────────────────────────────────────────────────┐
│  METHOD + URL                                       │
│  POST https://api.example.com/users                 │
├─────────────────────────────────────────────────────┤
│  HEADERS                                            │
│  Content-Type: application/json                     │
│  Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...     │
│  Accept: application/json                           │
├─────────────────────────────────────────────────────┤
│  BODY (optional, for POST/PUT/PATCH)                │
│  {                                                  │
│    "name": "Harshit",                               │
│    "email": "harshit@example.com"                   │
│  }                                                  │
└─────────────────────────────────────────────────────┘

                    ↓ Server processes ↓

┌─────────────────────────────────────────────────────┐
│  STATUS CODE                                        │
│  201 Created                                        │
├─────────────────────────────────────────────────────┤
│  RESPONSE HEADERS                                   │
│  Content-Type: application/json                     │
│  Location: /users/42                                │
├─────────────────────────────────────────────────────┤
│  RESPONSE BODY                                      │
│  {                                                  │
│    "id": 42,                                        │
│    "name": "Harshit",                               │
│    "email": "harshit@example.com",                  │
│    "created_at": "2026-03-17T10:30:00Z"             │
│  }                                                  │
└─────────────────────────────────────────────────────┘
```

**HTTP Methods (the verbs):**

| Method | Purpose | Has Body? | Idempotent? | Example |
|--------|---------|-----------|-------------|---------|
| GET | Read data | No | Yes | `GET /users/42` |
| POST | Create new data | Yes | No | `POST /users` |
| PUT | Replace existing data | Yes | Yes | `PUT /users/42` |
| PATCH | Partially update data | Yes | Yes | `PATCH /users/42` |
| DELETE | Remove data | Usually no | Yes | `DELETE /users/42` |

"Idempotent" means calling it multiple times has the same effect as calling it once. `DELETE /users/42` three times still results in user 42 being deleted. But `POST /users` three times creates three users. This matters for retry logic: you can safely retry GET, PUT, and DELETE. Retrying POST can create duplicates.

**What happens when you make an API call:** Your client (browser, app, server) opens a TCP connection to the server, sends the HTTP request, waits for the response, and closes the connection. HTTPS (the S is for secure) encrypts this entire exchange so nobody between you and the server can read or modify it.

**Under the hood:** The URL `https://api.example.com/users/42` breaks down into: protocol (`https`), domain (`api.example.com`, which DNS resolves to an IP address), and path (`/users/42`, which the server's router maps to a specific handler function).

### REST Conventions

REST isn't a specification or a protocol. It's a set of conventions. There's no REST police. But following these conventions makes your API predictable, which means any developer (or AI tool) can use it without reading docs for every endpoint.

**Resources are nouns, not verbs:**

```
Good (nouns):
  GET    /users          → list all users
  GET    /users/42       → get user 42
  POST   /users          → create a user
  PUT    /users/42       → update user 42
  DELETE /users/42       → delete user 42

Bad (verbs):
  GET    /getUsers
  POST   /createUser
  POST   /deleteUser/42
```

The HTTP method IS the verb. The URL is the noun. `GET /users` reads like "get users." `DELETE /users/42` reads like "delete user 42." You don't need the verb in the URL.

**Nested resources for relationships:**

```
GET  /users/42/orders         → all orders for user 42
GET  /users/42/orders/7       → order 7 for user 42
POST /users/42/orders         → create a new order for user 42
```

Don't nest deeper than two levels. `/users/42/orders/7/items/3/variations` is a nightmare. Instead, promote deeply nested resources to top-level: `GET /order-items/3`.

**Query parameters for filtering, sorting, and pagination:**

```
GET /users?role=admin                       → filter by role
GET /users?sort=created_at&order=desc       → sort by creation date
GET /users?page=2&limit=20                  → page 2, 20 per page
GET /users?role=admin&sort=name&limit=10    → combine them
```

Query parameters are for optional modifiers. If removing the parameter still returns a valid (though broader) response, it's a query parameter. If the value is required to identify the resource, it goes in the path (`/users/42`, not `/users?id=42`).

### Status Codes That Matter

You don't need to memorize all 60+ HTTP status codes. These 10 cover 99% of what you'll encounter:

**Success (2xx):**

| Code | Name | Meaning | When to use |
|------|------|---------|-------------|
| 200 | OK | Request succeeded, here's the data | GET, PUT, PATCH responses |
| 201 | Created | New resource created successfully | POST responses (return the created resource) |
| 204 | No Content | Success, but nothing to return | DELETE responses, or updates that return no body |

**Client errors (4xx) — your fault:**

| Code | Name | Meaning | When to use |
|------|------|---------|-------------|
| 400 | Bad Request | Request is malformed or invalid | Missing required fields, wrong data types, validation failures |
| 401 | Unauthorized | No valid authentication provided | Missing or expired auth token, bad API key |
| 403 | Forbidden | Authenticated but not allowed | User exists but doesn't have permission for this action |
| 404 | Not Found | Resource doesn't exist | Wrong URL, or the specific ID doesn't exist |
| 409 | Conflict | Request conflicts with current state | Duplicate email, trying to delete something that's referenced |
| 429 | Too Many Requests | Rate limit exceeded | Too many API calls in a given time window |

**Server errors (5xx) — their fault:**

| Code | Name | Meaning | When to use |
|------|------|---------|-------------|
| 500 | Internal Server Error | Something broke on the server | Unhandled exception, bug in server code |
| 503 | Service Unavailable | Server is temporarily down | Maintenance, overload, dependency failure |

**The key distinction:** 4xx means the client did something wrong (fix your request). 5xx means the server did something wrong (retry later or report the bug). This matters for error handling: you should retry 5xx and 429 errors. You should NOT retry 400, 401, 403, or 404 errors, because the same request will fail the same way.

### Request and Response Anatomy

**Headers you'll use constantly:**

```
Content-Type: application/json
  → "The body of my request is JSON"
  → Required for POST, PUT, PATCH requests with a JSON body

Accept: application/json
  → "Please send me JSON back"
  → Usually assumed, but explicit is better

Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
  → "Here's my auth token"
  → Goes in headers, NEVER in URL parameters

X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
  → "Unique ID for this request"
  → Useful for debugging: trace a request through multiple services
```

**A real request and response (JavaScript):**

```javascript
// Creating a new contact
const response = await fetch("https://api.example.com/contacts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`,
  },
  body: JSON.stringify({
    name: "Priya Sharma",
    email: "priya@example.com",
    company: "Acme Corp",
  }),
});

// Check status BEFORE parsing
if (!response.ok) {
  const error = await response.json();
  // error might be: { "message": "Email already exists", "field": "email" }
  throw new Error(`${response.status}: ${error.message}`);
}

const contact = await response.json();
// contact: { id: 42, name: "Priya Sharma", email: "priya@example.com", ... }
```

**A real request and response (Python):**

```python
import requests

response = requests.post(
    "https://api.example.com/contacts",
    headers={"Authorization": f"Bearer {token}"},
    json={
        "name": "Priya Sharma",
        "email": "priya@example.com",
        "company": "Acme Corp",
    },
)

response.raise_for_status()  # raises exception for 4xx/5xx
contact = response.json()
```

### Authentication Methods

**1. API Keys:** A static string that identifies your app. Simple, but limited.

```
# In a header (preferred)
Authorization: Bearer sk-1234567890abcdef

# In a query parameter (avoid — shows up in logs and browser history)
GET /data?api_key=sk-1234567890abcdef
```

**When to use:** Server-to-server calls, internal services, third-party APIs that don't involve user data (weather APIs, geocoding). Never expose API keys in frontend code.

**2. Bearer Tokens (JWT):** A token issued after login that proves who the user is. Sent with every request.

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0Mn0.abc123
```

The token contains encoded information (user ID, expiration time) and a signature that the server can verify. The server doesn't need to look anything up in a database. It just validates the signature.

**When to use:** User-facing apps where users log in. Most modern web apps use JWTs or session tokens.

**3. OAuth 2.0:** A flow for letting users grant your app access to their data on another service ("Sign in with Google," "Connect your GitHub").

```
1. Your app redirects user to Google's login page
2. User logs in and approves your app
3. Google redirects back to your app with a temporary code
4. Your server exchanges the code for an access token (server-to-server)
5. You use the access token to call Google APIs on behalf of the user
```

**When to use:** "Login with X" features, accessing user data on third-party platforms (reading their Google Calendar, posting to their Slack). Don't implement OAuth from scratch. Use a library.

**4. Session cookies:** The traditional approach. Server creates a session on login, stores it in a database, and sends a session ID as a cookie. The browser sends the cookie automatically with every request.

**When to use:** Traditional server-rendered web apps. Less common in API-first architectures but still valid.

### Pagination, Filtering, and Sorting

APIs that return lists need to handle large datasets. Nobody wants 10,000 records in a single response.

**Offset-based pagination (most common):**

```
GET /contacts?page=1&limit=20    → items 1-20
GET /contacts?page=2&limit=20    → items 21-40
GET /contacts?page=3&limit=20    → items 41-60
```

Response includes pagination metadata:

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 157,
    "total_pages": 8
  }
}
```

Simple but has a problem: if new items are inserted between page requests, items shift and you might see duplicates or miss items.

**Cursor-based pagination (better for real-time data):**

```
GET /contacts?limit=20                          → first 20
GET /contacts?limit=20&cursor=eyJpZCI6MjB9      → next 20 after cursor
```

The cursor is an opaque token (usually a base64-encoded ID or timestamp) that points to the last item you received. The server returns items after that point. No shifting, no duplicates.

```json
{
  "data": [...],
  "next_cursor": "eyJpZCI6NDB9",
  "has_more": true
}
```

**Filtering and sorting:**

```
GET /contacts?company=Acme&role=founder          → filter
GET /contacts?sort=created_at&order=desc          → sort
GET /contacts?search=priya                        → full-text search
GET /contacts?created_after=2026-01-01            → date range
```

### Rate Limiting

APIs limit how many requests you can make in a given time window. This prevents abuse, protects the server from overload, and ensures fair usage.

**How it works:**

```
Rate limit: 100 requests per minute

Request 1-100:   → 200 OK (with rate limit headers)
Request 101:     → 429 Too Many Requests

Response headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1679616000        (Unix timestamp when limit resets)
  Retry-After: 30                       (seconds to wait)
```

**Handling 429s with exponential backoff:**

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = response.headers.get("Retry-After");
      const waitMs = retryAfter
        ? parseInt(retryAfter) * 1000
        : Math.pow(2, attempt) * 1000 + Math.random() * 1000;

      console.log(`Rate limited. Retrying in ${waitMs}ms...`);
      await new Promise(resolve => setTimeout(resolve, waitMs));
      continue;
    }

    return response;
  }
  throw new Error("Max retries exceeded");
}
```

The jitter (`Math.random() * 1000`) prevents the "thundering herd" problem: if 100 clients all get rate-limited at the same time and all retry after exactly 2 seconds, they'll all hit the server simultaneously again.

## The Mental Model

**Mental model 1: "An API is a contract."** The URL says what resource you're talking about. The method says what you want to do. The status code says what happened. The body carries the data. If both sides (client and server) follow this contract, they can communicate without knowing anything about each other's internals. Your React frontend doesn't need to know the server uses PostgreSQL. The server doesn't need to know the client is a mobile app. The contract is the API.

**Mental model 2: "Every request is a stranger."** REST is stateless. The server treats every request as if it's never seen you before. There's no "session" on the server remembering your last request. If you need to prove who you are, you send your token every time. If you need context from a previous request, you include it. This feels redundant, but it's what makes APIs scalable: any server can handle any request because no server needs to remember anything.

**Mental model 3: "Status codes are a conversation."** 2xx means "yes." 4xx means "you did something wrong, here's what." 5xx means "I messed up, try later." Read the status code before you read the body. A 200 with error-shaped JSON is a code smell (and sadly common in poorly designed APIs). A proper API uses status codes to communicate the outcome, and the body to provide details.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| API | A way for one program to request things from another program | Every app, every integration |
| REST | A set of conventions for designing APIs using HTTP methods and URLs | Most backend services |
| Endpoint | A specific URL + method combination that does one thing | API documentation |
| Resource | The "thing" the API manages (a user, an order, a post) | URL design, data modeling |
| HTTP method | The verb (GET, POST, PUT, DELETE) that says what you want to do | Every API request |
| Status code | A number the server sends back saying what happened (200 = OK, 404 = not found) | Every API response |
| Header | Metadata sent with a request or response (auth token, content type) | Authentication, content negotiation |
| Body / Payload | The actual data sent in a request or response (usually JSON) | POST/PUT requests, all responses with data |
| JSON | JavaScript Object Notation, the standard data format for APIs | Almost every modern API |
| Idempotent | Calling it multiple times has the same effect as calling it once | PUT, DELETE, GET (not POST) |
| Stateless | Server doesn't remember previous requests | REST architecture |
| Bearer token | A string you send in the Authorization header to prove who you are | Authenticated API calls |
| Rate limit | Max number of requests allowed in a time window | API usage at scale |
| Pagination | Breaking large lists into pages so you don't return everything at once | Any list endpoint |
| Cursor | An opaque pointer to a position in a list, used for pagination | Real-time or large-scale APIs |
| Webhook | An API in reverse: the server calls YOUR URL when something happens | Notifications, integrations |
| CORS | Browser security that blocks your frontend from calling APIs on different domains | Frontend development |
| Idempotency key | A unique ID sent with a POST to prevent duplicate creation on retries | Payment APIs, critical operations |

## Common Patterns

### 1. Fetching Data for a Frontend

**When to use it:** Your React/Next.js app needs to load and display data from a backend.

**How it works:** GET request on mount (or with a data-fetching library), parse the JSON response, handle loading/error/empty states.

**Code sketch:**

```typescript
// Using fetch in a Next.js Server Component
async function ContactsPage() {
  const response = await fetch("https://api.example.com/contacts?limit=20", {
    headers: { Authorization: `Bearer ${getToken()}` },
  });

  if (!response.ok) {
    throw new Error(`Failed to load contacts: ${response.status}`);
  }

  const { data: contacts, pagination } = await response.json();

  return (
    <div>
      {contacts.map(contact => (
        <ContactCard key={contact.id} contact={contact} />
      ))}
      {pagination.has_more && <LoadMoreButton cursor={pagination.next_cursor} />}
    </div>
  );
}
```

```typescript
// Client-side with TanStack Query
function useContacts(page: number) {
  return useQuery({
    queryKey: ["contacts", page],
    queryFn: async () => {
      const res = await fetch(`/api/contacts?page=${page}&limit=20`);
      if (!res.ok) throw new Error(`${res.status}`);
      return res.json();
    },
  });
}
```

### 2. Submitting Forms

**When to use it:** User fills out a form and you need to send the data to create or update a resource.

**How it works:** POST (create) or PUT/PATCH (update) with the form data as JSON in the body. Handle validation errors from the server.

**Code sketch:**

```typescript
async function createContact(formData: ContactForm): Promise<Contact> {
  const response = await fetch("/api/contacts", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      name: formData.name,
      email: formData.email,
      company: formData.company,
    }),
  });

  if (response.status === 400) {
    // Server returned validation errors
    const { errors } = await response.json();
    // errors: { email: "Email already exists" }
    throw new ValidationError(errors);
  }

  if (!response.ok) {
    throw new Error(`Failed to create contact: ${response.status}`);
  }

  return response.json(); // returns the created contact with ID
}
```

### 3. File Uploads

**When to use it:** Users need to upload images, documents, or other files.

**How it works:** Use `multipart/form-data` instead of JSON. The file is sent as binary data alongside any metadata fields.

**Code sketch:**

```typescript
async function uploadAvatar(userId: string, file: File) {
  const formData = new FormData();
  formData.append("avatar", file);
  formData.append("user_id", userId);

  const response = await fetch("/api/uploads/avatar", {
    method: "POST",
    // Don't set Content-Type header — the browser sets it
    // automatically with the correct multipart boundary
    headers: { Authorization: `Bearer ${token}` },
    body: formData,
  });

  if (!response.ok) throw new Error(`Upload failed: ${response.status}`);
  return response.json(); // { url: "https://cdn.example.com/avatars/42.jpg" }
}
```

Important: do NOT set `Content-Type: application/json` for file uploads. Don't set `Content-Type` at all. The browser adds the correct `multipart/form-data` header with a boundary string automatically. Setting it manually breaks the upload.

### 4. Webhook Receivers

**When to use it:** An external service needs to notify your app when something happens (Stripe payment completed, GitHub push, Telegram message received).

**How it works:** You expose a POST endpoint. The external service calls it with event data. You process the event and return 200 to acknowledge receipt.

**Code sketch:**

```python
# FastAPI webhook receiver for Stripe
from fastapi import FastAPI, Request, HTTPException
import hmac
import hashlib

app = FastAPI()

@app.post("/webhooks/stripe")
async def stripe_webhook(request: Request):
    body = await request.body()
    signature = request.headers.get("Stripe-Signature", "")

    # ALWAYS verify the webhook signature
    if not verify_stripe_signature(body, signature, WEBHOOK_SECRET):
        raise HTTPException(status_code=401, detail="Invalid signature")

    event = json.loads(body)

    if event["type"] == "payment_intent.succeeded":
        payment = event["data"]["object"]
        await handle_successful_payment(payment)
    elif event["type"] == "payment_intent.failed":
        await handle_failed_payment(event["data"]["object"])

    # Return 200 quickly — do heavy processing in background
    return {"received": True}
```

Two critical rules for webhooks: always verify the signature (anyone can POST to your URL), and always return 200 quickly (the sender will retry if you take too long, creating duplicates).

### 5. API Wrapper Services

**When to use it:** Your frontend needs data from a third-party API, but you can't call it directly from the browser (CORS, API key exposure, or you need to transform the data).

**How it works:** Your backend acts as a proxy. Frontend calls your API, your API calls the third-party API, transforms the response, and returns it.

**Code sketch:**

```python
# Your backend wraps a third-party weather API
@app.get("/api/weather/{city}")
async def get_weather(city: str):
    # Your API key stays on the server, never exposed to frontend
    response = requests.get(
        f"https://api.weatherapi.com/v1/current.json",
        params={"key": WEATHER_API_KEY, "q": city},
        timeout=5,
    )

    if response.status_code == 404:
        raise HTTPException(status_code=404, detail=f"City '{city}' not found")

    if not response.ok:
        raise HTTPException(status_code=502, detail="Weather service unavailable")

    data = response.json()

    # Transform: return only what the frontend needs
    return {
        "city": data["location"]["name"],
        "temp_c": data["current"]["temp_c"],
        "condition": data["current"]["condition"]["text"],
        "icon": data["current"]["condition"]["icon"],
    }
```

This pattern keeps your API key safe, lets you transform data to match your frontend's needs, and lets you switch providers without changing frontend code.

## Mistakes Everyone Makes

### 1. Not handling errors (assuming every request succeeds)

**What people do:** Call `response.json()` without checking `response.ok` first.

**Why it seems right:** "It works in my testing."

**What actually happens:** In production, APIs fail. Networks drop. Servers go down. Tokens expire. Your code tries to parse an error HTML page as JSON and throws a cryptic `SyntaxError: Unexpected token '<'`. Or it treats a 500 error body as valid data and displays garbage to the user. The app looks broken with no useful error message.

**What to do instead:** Always check the status code before parsing the response. Handle different error codes differently: 401 means redirect to login, 404 means show a "not found" state, 429 means retry, 500 means show a generic error message and log it.

```typescript
const response = await fetch(url);

if (response.status === 401) {
  redirectToLogin();
  return;
}

if (!response.ok) {
  const errorBody = await response.text();
  console.error(`API error ${response.status}:`, errorBody);
  throw new Error(`Request failed: ${response.status}`);
}

const data = await response.json();
```

### 2. Sending sensitive data in URL parameters

**What people do:** `GET /users?api_key=sk-secret123` or `GET /login?password=hunter2`.

**Why it seems right:** "It's HTTPS, it's encrypted."

**What actually happens:** URLs show up in browser history, server access logs, proxy logs, analytics tools, error tracking services, and Referer headers when you click links. HTTPS encrypts data in transit, but the URL is stored in plain text in dozens of places. Your API key ends up in Datadog, in Nginx logs, in the user's browser history, and eventually on a compromised laptop.

**What to do instead:** Send sensitive data in request headers (`Authorization: Bearer token`) or in the request body (for POST). Never in the URL. This includes tokens, passwords, API keys, and any personally identifiable information.

### 3. Ignoring rate limits

**What people do:** Fire requests in a tight loop. When they get 429s, they ignore them or crash.

**Why it seems right:** "I need all this data, so I'll request it all as fast as possible."

**What actually happens:** The API blocks your requests. If you keep hammering, some APIs will temporarily or permanently ban your API key. Your batch job fails halfway through. You have to start over.

**What to do instead:** Read the rate limit headers (`X-RateLimit-Remaining`). When you get a 429, wait for the duration specified in the `Retry-After` header. For batch processing, throttle your requests to stay under the limit. Use a concurrency limiter (semaphore) for parallel requests.

### 4. Not validating response shape before using it

**What people do:** `const name = response.data.user.profile.name` without checking if any of those nested properties exist.

**Why it seems right:** "The API documentation says it returns this shape."

**What actually happens:** The API changes its response format. Or it returns a different shape for edge cases (empty arrays, null values, missing optional fields). Your code throws `Cannot read property 'name' of undefined` and the whole page crashes.

**What to do instead:** Validate the response with Zod or a similar library. At minimum, use optional chaining (`response.data?.user?.profile?.name`) and provide fallbacks. For critical code paths, validate the full schema.

```typescript
import { z } from "zod";

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  company: z.string().nullable(),
});

const raw = await response.json();
const user = UserSchema.parse(raw); // throws if shape doesn't match
```

### 5. Building APIs around database tables instead of client needs

**What people do:** Create one API endpoint per database table: `/users`, `/orders`, `/order_items`, `/products`. Then the frontend needs to make 4 requests and join the data in JavaScript.

**Why it seems right:** "It's clean and follows REST."

**What actually happens:** The frontend makes N+1 requests: one to get the order, then one per order item to get product details. The page is slow. The code is complex. The frontend developer writes a join function in JavaScript that's worse than what the database could do in one query.

**What to do instead:** Design APIs around what the client needs, not how your database is structured. If the "order details" page needs order info, items, and product names, return it all in one endpoint: `GET /orders/42/details`. This is the BFF (Backend for Frontend) pattern. Your API is an interface for your client, not a mirror of your schema.

### 6. Not setting timeouts

**What people do:** Call `fetch(url)` or `requests.get(url)` without a timeout.

**Why it seems right:** "The server will respond eventually."

**What actually happens:** The server hangs. Your request waits. In a browser, the user stares at a spinner forever. On a server, you run out of connections. In a serverless function, you hit the execution timeout and pay for wasted compute.

**What to do instead:** Always set timeouts. 5-10 seconds for most API calls. 30 seconds for file uploads. Shorter is better.

```python
# Python
response = requests.get(url, timeout=5)  # 5 second timeout

# JavaScript
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000);
const response = await fetch(url, { signal: controller.signal });
clearTimeout(timeout);
```

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Always check `response.ok` (or the status code) before parsing the body.** Never assume a request succeeded. Handle 401, 403, 404, 429, and 5xx differently.

2. **Put auth tokens in headers, never in URLs.** URLs get logged, cached, and shared. Headers don't. Use `Authorization: Bearer <token>` for every authenticated request.

3. **Set timeouts on every outgoing request.** 5 seconds for normal API calls, 30 seconds for uploads. A missing timeout is a memory leak waiting to happen.

4. **Validate response shapes with a schema library (Zod, Pydantic).** APIs change. Optional fields get removed. New fields appear. Validation catches these at the boundary instead of deep in your rendering code.

5. **Use exponential backoff with jitter for retries.** Only retry 429 and 5xx. Never retry 400 or 401. Add randomness to prevent thundering herds.

6. **Design API endpoints around client needs, not database tables.** One endpoint that returns everything the page needs is better than five endpoints the frontend has to stitch together.

7. **Return consistent error responses.** Every error should be JSON with at least a `message` field. Include a `code` for programmatic handling and `field` for validation errors. Never return raw stack traces or HTML error pages from an API.

8. **Log request IDs end to end.** Generate a unique ID for each request, include it in all logs, and return it in the response. When something breaks, this ID lets you trace the request through every service it touched.

## The "Just Tell Me What to Do" Quickstart

Build a working REST API with FastAPI in under 10 minutes. CRUD for a "contacts" resource with proper status codes, validation, and error handling.

**Step 1: Install FastAPI**

```bash
pip install fastapi uvicorn pydantic
```

**Step 2: Create `main.py`**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr
from datetime import datetime

app = FastAPI()

# In-memory database (replace with a real DB later)
contacts: dict[int, dict] = {}
next_id = 1


# --- Schemas ---

class ContactCreate(BaseModel):
    name: str
    email: str
    company: str | None = None
    phone: str | None = None

class ContactUpdate(BaseModel):
    name: str | None = None
    email: str | None = None
    company: str | None = None
    phone: str | None = None


# --- Endpoints ---

@app.get("/contacts")
def list_contacts(limit: int = 20, offset: int = 0):
    """List all contacts with pagination."""
    all_contacts = list(contacts.values())
    return {
        "data": all_contacts[offset : offset + limit],
        "total": len(all_contacts),
        "limit": limit,
        "offset": offset,
    }


@app.get("/contacts/{contact_id}")
def get_contact(contact_id: int):
    """Get a single contact by ID."""
    if contact_id not in contacts:
        raise HTTPException(status_code=404, detail=f"Contact {contact_id} not found")
    return contacts[contact_id]


@app.post("/contacts", status_code=201)
def create_contact(contact: ContactCreate):
    """Create a new contact."""
    global next_id

    # Check for duplicate email
    for existing in contacts.values():
        if existing["email"] == contact.email:
            raise HTTPException(
                status_code=409,
                detail={"message": "Email already exists", "field": "email"},
            )

    new_contact = {
        "id": next_id,
        **contact.model_dump(),
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat(),
    }
    contacts[next_id] = new_contact
    next_id += 1
    return new_contact


@app.put("/contacts/{contact_id}")
def update_contact(contact_id: int, updates: ContactUpdate):
    """Update a contact."""
    if contact_id not in contacts:
        raise HTTPException(status_code=404, detail=f"Contact {contact_id} not found")

    contact = contacts[contact_id]
    update_data = updates.model_dump(exclude_unset=True)

    if not update_data:
        raise HTTPException(status_code=400, detail="No fields to update")

    contact.update(update_data)
    contact["updated_at"] = datetime.now().isoformat()
    return contact


@app.delete("/contacts/{contact_id}", status_code=204)
def delete_contact(contact_id: int):
    """Delete a contact."""
    if contact_id not in contacts:
        raise HTTPException(status_code=404, detail=f"Contact {contact_id} not found")
    del contacts[contact_id]
```

**Step 3: Run it**

```bash
uvicorn main:app --reload
```

**Step 4: Test it**

Open `http://localhost:8000/docs` in your browser. FastAPI generates interactive API docs automatically. Click any endpoint, click "Try it out," fill in the fields, and click "Execute."

Or use curl:

```bash
# Create a contact
curl -X POST http://localhost:8000/contacts \
  -H "Content-Type: application/json" \
  -d '{"name": "Priya Sharma", "email": "priya@example.com", "company": "Acme"}'

# List contacts
curl http://localhost:8000/contacts

# Get one contact
curl http://localhost:8000/contacts/1

# Update a contact
curl -X PUT http://localhost:8000/contacts/1 \
  -H "Content-Type: application/json" \
  -d '{"company": "New Company"}'

# Delete a contact
curl -X DELETE http://localhost:8000/contacts/1
```

**Step 5: Try breaking it**

- POST with a duplicate email. You should get a 409.
- GET a contact that doesn't exist. You should get a 404.
- PUT with an empty body. You should get a 400.
- POST without the required `name` field. FastAPI's validation returns a 422.

This is what a well-behaved API looks like: clear status codes, validation, and error messages.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to build APIs, always include:
- The framework (FastAPI, Express, Hono, Next.js API routes)
- The database (PostgreSQL, SQLite, Supabase, in-memory for prototypes)
- What resources the API manages (users, orders, contacts)
- Authentication approach (API key, JWT, session, none for internal)
- Whether this is an internal API or public-facing (public needs rate limiting, versioning, better error messages)

### Example prompts that produce good results

**For building a CRUD API:**
```
Build a FastAPI REST API for managing "tasks". Each task has: id (auto),
title (required string), description (optional string), status (enum:
todo/in_progress/done, default: todo), created_at, updated_at.

Endpoints: GET /tasks (with status filter and pagination), GET /tasks/{id},
POST /tasks, PATCH /tasks/{id}, DELETE /tasks/{id}.

Use Pydantic for validation. Return proper status codes (201 for create,
204 for delete, 404 for not found). Include error handling.
```

**For adding authentication:**
```
Add JWT authentication to my FastAPI app. I need: POST /auth/login that
takes email + password and returns a JWT, a dependency that extracts and
validates the JWT from the Authorization header, and protection on all
endpoints except login. Use python-jose for JWT handling. Tokens expire
after 24 hours.
```

**For debugging API issues:**
```
My frontend fetch call gets a CORS error when calling my FastAPI backend
on a different port. The error is: "Access-Control-Allow-Origin header
is missing." How do I configure CORS in FastAPI to allow requests from
localhost:3000? Show me the middleware setup.
```

### What to watch out for in AI-generated API code

- **Missing validation:** AI often skips input validation for optional fields or doesn't check string lengths. Always verify that validation is thorough.
- **Wrong status codes:** AI commonly returns 200 for everything. Check that create returns 201, delete returns 204, and errors return appropriate 4xx codes.
- **No error handling for database calls:** AI wraps the happy path but forgets to handle what happens when a database query fails.
- **CORS not configured:** If your frontend and backend are on different domains/ports (they usually are in development), AI might not add CORS middleware.
- **SQL injection in raw queries:** If the AI writes raw SQL instead of using an ORM, check that it uses parameterized queries, not string interpolation.

### Key terms that improve output quality

- "Use proper HTTP status codes" (prevents the 200-for-everything pattern)
- "Add Pydantic validation" or "Add Zod validation" (gets input validation)
- "Include error responses in the OpenAPI schema" (gets documented error formats)
- "Add pagination to list endpoints" (prevents returning unbounded data)
- "Return consistent error format" (gets uniform error responses)

## Ship It: Build This

### Contacts REST API with Full CRUD

Build a complete REST API for managing contacts, with proper status codes, input validation, error handling, pagination, and filtering. This is the backend you'd pair with any CRM or address book frontend.

**Why this project:** It exercises every concept in this guide. CRUD operations map to HTTP methods. Validation catches bad input. Pagination handles large datasets. Error responses communicate what went wrong. You'll refer back to this API as a template every time you build a backend.

**Rough architecture:**
- FastAPI (Python) or Express (Node.js), your choice
- SQLite database (no setup needed, file-based)
- Pydantic or Zod for input validation
- Five endpoints: list (with search, sort, pagination), get by ID, create, update, delete
- Proper status codes: 200, 201, 204, 400, 404, 409, 500
- Consistent error response format: `{ "error": { "code": "...", "message": "...", "field": "..." } }`
- Request logging middleware (method, path, status code, duration)
- Auto-generated API docs (FastAPI does this, Express needs swagger-jsdoc)

**Key features to build:**
- CRUD with all proper status codes
- Input validation (required fields, email format, string lengths)
- Duplicate detection (no two contacts with the same email)
- Search by name or email (`?search=priya`)
- Sort by any field (`?sort=created_at&order=desc`)
- Pagination (`?page=1&limit=20` with total count in response)
- Request timing middleware
- Error handling that never leaks stack traces

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn GraphQL** when your frontend needs flexible queries (different pages need different fields from the same resource) and you're tired of creating a new endpoint for each use case.
- **Learn API versioning** when your API has external consumers and you need to make breaking changes without breaking their code. Start with URL versioning (`/v1/users`) for simplicity.
- **Learn OpenAPI/Swagger** when you need auto-generated documentation, client SDKs, or contract-first API design. FastAPI generates this automatically. For Express, use swagger-jsdoc.
- **Learn gRPC** when you need high-performance server-to-server communication with strict schemas and streaming support. Not for browser clients (use REST or GraphQL for that).
- **Learn HATEOAS** when you're building a truly RESTful API that tells clients what they can do next by including links in responses. Most APIs skip this and nobody misses it, but it's worth understanding the theory.
