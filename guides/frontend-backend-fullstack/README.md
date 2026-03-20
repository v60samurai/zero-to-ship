# Frontend vs Backend vs Fullstack: Zero to Ship

> The dining room, the kitchen, and the waiter who connects them.

## Why Should You Care?

When you build an app, code runs in two different places: on the user's device (frontend) and on a server somewhere (backend). Understanding this split is fundamental to building anything on the web. You need to know which side handles what, where your data lives, and how the two sides talk to each other.

If you're using AI tools to build, you'll constantly hear "that's a frontend issue" or "you need a backend for that." Without understanding the boundary, you'll ask AI to do impossible things, like "store user data in React" (React runs in the browser; it can't permanently store anything). You'll get confused when someone says "add an API endpoint" because you won't know where that code goes or how the frontend calls it.

This guide gives you the mental model. By the end, you'll understand where code runs, why the frontend and backend are separated, how they communicate, and what "fullstack" actually means. You'll know why some changes are "quick frontend tweaks" and others require "backend work," and you'll be able to have informed conversations about architecture decisions.

## The 30-Second Version

The **frontend** is what runs in the user's browser: the buttons they click, the pages they see, the forms they fill out. The **backend** is what runs on a server: receiving requests, checking permissions, reading from the database, applying business logic, and sending responses. They communicate through **APIs**, which are just agreed-upon ways to send requests and receive data. The **database** stores data permanently; only the backend talks to it directly. **Fullstack** means handling both frontend and backend, often in a single codebase. Modern frameworks like Next.js blur the line by letting you write frontend and backend code together.

## The Real Explanation (ELI5)

Imagine a restaurant.

**The dining room** is the frontend. It's what customers see and interact with. The decor, the tables, the menu, the way dishes are presented. Customers sit down, read the menu, and tell the waiter what they want. They never go into the kitchen. They don't know how the food is made. They just see the final result on their plate.

**The kitchen** is the backend. It's where the actual work happens. Chefs receive orders, pull ingredients from the pantry, follow recipes, cook the food, and plate it. The kitchen has rules: only certain people can access certain ingredients, orders are prepared in a specific sequence, and food must meet quality standards before it goes out. Customers never see any of this.

**The waiter** is the API. The waiter takes orders from customers (requests) and carries them to the kitchen. The waiter brings food back from the kitchen (responses) and delivers it to the right table. The waiter doesn't cook. The waiter doesn't decide what's on the menu. The waiter just carries messages back and forth in a specific format everyone agrees on.

**The pantry** is the database. It's where ingredients (data) are stored permanently. Only kitchen staff access the pantry. If a customer wants to know if something is in stock, they ask the waiter, who asks the kitchen, who checks the pantry.

Now replace "dining room" with "browser," "kitchen" with "server," "waiter" with "HTTP API," and "pantry" with "database." That's web development.

The analogy is accurate at one critical point: customers (users) cannot directly access the kitchen (server) or pantry (database). Everything goes through the waiter (API). This separation exists for security (customers shouldn't roam the kitchen), efficiency (the kitchen can serve many dining rooms), and organization (front-of-house and back-of-house have different skills).

The analogy breaks down when we talk about modern fullstack frameworks. It's like if the waiter could also do some cooking at a station in the dining room, or if some menu items were pre-prepared and didn't need the kitchen at all. That's server-side rendering and static generation, and we'll cover that later.

## How It Actually Works

### Frontend: What Runs in the Browser

When someone visits your website, their browser downloads files and runs them locally. These files are the frontend.

**HTML** defines structure: what elements exist on the page.
```html
<h1>Welcome</h1>
<p>This is a paragraph.</p>
<button>Click me</button>
```

**CSS** defines styling: how elements look.
```css
h1 { color: blue; font-size: 24px; }
button { background: green; padding: 10px; }
```

**JavaScript** defines behavior: what happens when users interact.
```javascript
document.querySelector('button').addEventListener('click', () => {
  alert('You clicked!');
});
```

Every website, no matter how complex, is built from these three: HTML, CSS, and JavaScript.

**Why frameworks exist (React, Vue, Next.js):**

Building complex UIs with raw HTML/CSS/JS becomes painful quickly. Imagine a dashboard with 50 interactive components, each needing to update when data changes. Manually updating the DOM (the browser's representation of HTML) leads to bugs and spaghetti code.

Frameworks like React solve this by letting you describe what the UI should look like for a given state, and the framework handles updating the DOM when state changes:

```jsx
// React component
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

You say "when count is 5, show 5." React figures out how to update the DOM efficiently. This is called declarative UI.

**What the frontend CAN do:**
- Display content and styling
- Respond to user interactions (clicks, typing, scrolling)
- Make requests to APIs
- Store temporary data (in memory, localStorage, sessionStorage)
- Run complex logic and calculations
- Animate and transition elements

**What the frontend CANNOT do:**
- Permanently store data (localStorage persists but users can clear it)
- Keep secrets (all frontend code is visible to users)
- Directly access databases
- Send emails
- Process payments securely
- Anything that requires trusting the code (users can modify frontend code)

### Backend: What Runs on the Server

The backend is code that runs on a server you control. Users never see this code; they only see what it sends back.

A backend server typically:
1. Listens for incoming HTTP requests
2. Parses the request (what URL? what method? what data?)
3. Authenticates and authorizes (who is this? are they allowed?)
4. Executes business logic (the actual work)
5. Reads from or writes to the database
6. Sends back a response

**Example in Node.js (Express):**

```javascript
const express = require('express');
const app = express();

// This code runs on the server, not in any browser
app.get('/api/users/:id', async (req, res) => {
  // 1. Parse request: user wants user with this ID
  const userId = req.params.id;

  // 2. Check authentication (simplified)
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Not authenticated' });
  }

  // 3. Check authorization
  if (!canAccessUser(req.user, userId)) {
    return res.status(403).json({ error: 'Not authorized' });
  }

  // 4. Business logic + database query
  const user = await database.query('SELECT * FROM users WHERE id = $1', [userId]);

  // 5. Send response
  res.json(user);
});

app.listen(3000);
```

**Common backend languages and frameworks:**

| Language | Framework | Notes |
|----------|-----------|-------|
| JavaScript | Node.js + Express/Fastify | Same language as frontend, huge ecosystem |
| Python | FastAPI, Django, Flask | Great for data/ML, clean syntax |
| Go | Standard library, Gin, Echo | Fast, good for high-performance APIs |
| Ruby | Rails | Convention over configuration, rapid development |
| Java | Spring Boot | Enterprise standard, verbose but robust |
| Rust | Axum, Actix | Maximum performance, steeper learning curve |

**What the backend CAN do:**
- Store data permanently in databases
- Keep secrets (API keys, encryption keys)
- Enforce business rules that users can't bypass
- Send emails, process payments, call external APIs
- Heavy computation without slowing down the user's browser
- Anything that requires trust (users can't modify server code)

**What the backend CANNOT do:**
- Directly update what users see (it can only send data; the frontend renders it)
- Run without infrastructure (needs a server, which costs money and requires management)

### The Database: Where Data Lives

The database stores data permanently. When a user creates an account, that data goes in the database. When they log in tomorrow, the backend retrieves it from the database.

```
Frontend: "Show me my profile"
    │
    ▼
Backend: Receives request, queries database
    │
    ▼
Database: "Here's the row for user 123"
    │
    ▼
Backend: Formats response, sends to frontend
    │
    ▼
Frontend: Renders the profile page
```

**Why the frontend never talks to the database directly:**

1. **Security:** Database credentials would be in frontend code. Anyone could read them.
2. **Authorization:** The frontend can't enforce "users can only see their own data."
3. **Validation:** Users could send malicious queries directly to the database.
4. **Abstraction:** If you change databases, every frontend client would break.

The backend is the gatekeeper. It checks permissions, validates input, and only then queries the database.

**Common database types:**

| Type | Examples | Best For |
|------|----------|----------|
| Relational (SQL) | PostgreSQL, MySQL, SQLite | Structured data with relationships |
| Document (NoSQL) | MongoDB, Firestore | Flexible schemas, JSON-like data |
| Key-Value | Redis, DynamoDB | Caching, sessions, simple lookups |
| Vector | Pinecone, Weaviate | AI embeddings, similarity search |

For most apps, PostgreSQL is the default choice. See the [Databases and SQL guide](../databases-sql/) for more.

### APIs: The Boundary Between Frontend and Backend

An API (Application Programming Interface) is how the frontend and backend communicate. In web development, this almost always means HTTP requests with JSON data.

**The flow:**

```
Frontend                              Backend
   │                                     │
   │─── POST /api/login ────────────────▶│
   │    {"email": "a@b.com",             │
   │     "password": "secret"}           │
   │                                     │
   │                                     │ Validate credentials
   │                                     │ Check database
   │                                     │ Create session
   │                                     │
   │◀── 200 OK ──────────────────────────│
   │    {"user": {"id": 1, "name": "A"}, │
   │     "token": "abc123"}              │
   │                                     │
```

**JSON is the common language:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"}
  ],
  "total": 2,
  "page": 1
}
```

Both frontend JavaScript and backend code can easily parse and generate JSON.

**Why separate frontend and backend?**

1. **Security boundary:** Secrets stay on the server. Business rules are enforced server-side.
2. **Multiple clients:** One backend can serve a web app, mobile app, and third-party integrations.
3. **Independent scaling:** High traffic to the frontend? Cache it on a CDN. Slow database queries? Scale the backend separately.
4. **Team specialization:** Frontend developers focus on UI/UX. Backend developers focus on data and logic.
5. **Technology flexibility:** Frontend in React, backend in Go, database in PostgreSQL. Best tool for each job.

### Fullstack: Both Sides, One Person or Framework

"Fullstack" means working on both frontend and backend. A fullstack developer can build the whole application. A fullstack framework lets you write both in one codebase.

**Next.js as an example:**

Next.js is a React framework that includes backend capabilities:

```
my-app/
├── app/
│   ├── page.tsx           ← Frontend: React component
│   ├── api/
│   │   └── users/
│   │       └── route.ts   ← Backend: API endpoint
│   └── layout.tsx
└── lib/
    └── db.ts              ← Database connection
```

**Frontend component (runs in browser):**

```tsx
// app/page.tsx
'use client';

import { useState, useEffect } from 'react';

export default function Home() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => setUsers(data));
  }, []);

  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**Backend API route (runs on server):**

```typescript
// app/api/users/route.ts
import { db } from '@/lib/db';

export async function GET() {
  const users = await db.query('SELECT id, name FROM users');
  return Response.json(users);
}
```

Same codebase, same language, but they run in completely different places.

**Why the line is blurring:**

Modern frameworks offer:
- **Server Components:** React components that run on the server, not in the browser
- **Server Actions:** Functions that run on the server, called directly from frontend code
- **Edge Functions:** Backend code that runs at CDN edge locations, closer to users

This reduces the mental overhead of "frontend vs backend" for many use cases, while still maintaining the security boundary where it matters.

### Server-Side Rendering vs Client-Side Rendering

Where HTML gets generated matters for performance and SEO.

**Client-Side Rendering (CSR):**

```
1. Browser requests page
2. Server sends minimal HTML + JavaScript bundle
3. Browser downloads and executes JavaScript
4. JavaScript fetches data from API
5. JavaScript renders the actual content
```

The user sees a blank page (or loading spinner) until steps 3-5 complete. Search engines may not see the content if they don't execute JavaScript.

**Server-Side Rendering (SSR):**

```
1. Browser requests page
2. Server runs the code, fetches data, generates full HTML
3. Server sends complete HTML
4. Browser displays content immediately
5. JavaScript "hydrates" to make it interactive
```

The user sees content faster. Search engines see the full page.

**Static Site Generation (SSG):**

```
1. At build time: generate HTML for all pages
2. Deploy HTML files to CDN
3. Browser requests page
4. CDN serves pre-built HTML instantly
```

Fastest possible delivery; pages are just files. But content is fixed until you rebuild.

**When to use what:**

| Approach | Best For | Tradeoff |
|----------|----------|----------|
| CSR | Dashboards, apps behind login | Slower initial load, no SEO needed |
| SSR | Dynamic content needing SEO | Server cost, complexity |
| SSG | Blogs, marketing sites, docs | Content fixed at build time |

Modern frameworks (Next.js, Remix, Nuxt) let you mix these per-page. Your marketing homepage can be static, your product pages can be SSR, and your dashboard can be CSR.

**Common confusion:** "Server-side rendering" is NOT the same as "backend." SSR is about where the HTML is generated. The backend is about where business logic and data live. You can have SSR without a traditional backend (static site with pre-rendered pages). You can have a backend without SSR (API that serves JSON to a CSR frontend).

### Where Things Actually Run

Let's get concrete about physical locations.

**Browser:**
- Runs on the user's device (laptop, phone)
- You send code; their device executes it
- You have zero control over the environment (browser version, extensions, network speed)
- All code and data in the browser is visible to the user

**Traditional Server:**
- A computer in a data center running 24/7
- You control everything: OS, runtime, dependencies
- Runs until you stop it or it crashes
- You pay for it whether it's being used or not

**Serverless Functions:**
- Code that runs on-demand, spins up for a request, then goes away
- AWS Lambda, Vercel Functions, Cloudflare Workers
- You pay per execution, not for idle time
- Cold starts: first request after inactivity is slower

**Edge:**
- Servers distributed globally, close to users
- Cloudflare Workers, Vercel Edge Functions, Deno Deploy
- Lower latency (response from nearby server)
- Limited runtime (can't run heavy computation or long database queries)

**Visual:**

```
User in Tokyo
     │
     ▼
[Edge Server in Tokyo]         ← Edge function responds (10ms)
     │
     ▼ (if edge can't handle it)
[Origin Server in US-East]     ← Traditional backend (150ms)
     │
     ▼
[Database in US-East]          ← Data read (20ms)
```

**Practical mapping:**

| What | Where It Runs |
|------|---------------|
| React components with `'use client'` | Browser |
| Next.js API routes | Server (or serverless) |
| Next.js Server Components | Server (at request time or build time) |
| Vercel Edge Functions | Edge locations globally |
| Your PostgreSQL database | A specific server/region |
| Static assets (JS, CSS, images) | CDN (cached globally) |

### The PM Perspective: What Changes Actually Mean

When engineers say "that's a frontend change" or "we need backend work," here's what they mean.

**"It's just a frontend change":**

- The data already exists and is already being sent to the frontend
- We're changing how it's displayed, not what it is
- Examples: changing button color, reordering columns, adding a tooltip
- Usually faster: change the code, deploy, done
- Caveat: "Just change the UI" can still be complex if the UI is complex

**"We need a backend change":**

- The data doesn't exist yet, or it's not being sent to the frontend
- We need to: add a database column, create a new API endpoint, modify business logic
- Examples: adding a new field to user profiles, implementing search, changing pricing logic
- Usually slower: database migration, API changes, update frontend to use new API, testing
- Caveat: some "backend changes" are trivial (add one field), others are architectural

**"That needs a new API endpoint":**

- The frontend wants to do something the backend doesn't support yet
- Common when features require new data or new actions
- Timeline depends on complexity of the logic, not just "making an endpoint"

**"We need to update the database schema":**

- Data model is changing (new tables, new columns, new relationships)
- Requires migration strategy (what about existing data?)
- Highest risk: database changes can break things if not done carefully

**Red flags that a "simple change" isn't simple:**

1. "Just add a filter" → might need a new database index, new API parameter, frontend state management
2. "Show this data on the frontend" → data might not exist, or not be accessible for security reasons
3. "Let users edit this" → need to handle validation, permissions, optimistic updates, error states
4. "Make it faster" → could be frontend (rendering), backend (query), database (index), or all three

**Questions to ask:**

- Does the data exist? (Is it in the database?)
- Is the data exposed? (Is there an API for it?)
- Is the data authorized? (Can this user see it?)
- What changes if we're wrong? (Undo-ability)

## The Mental Model

**Think of the browser as a stranger's computer that you're sending instructions to.** You can't trust it to keep secrets. You can't trust it to enforce rules. You can only trust it to display things and send requests back. This means whenever you're building something sensitive (payments, authentication, business logic), ask: "What happens if the user modifies this code?"

**Think of the API as a contract.** The frontend says "I'll send you this shape of data" and the backend says "I'll respond with this shape of data." Both sides can change internally without breaking the contract. This means whenever you're debugging, check the contract: is the frontend sending what the backend expects? Is the backend returning what the frontend expects?

**Think of fullstack as choosing which side executes your code.** Some code must run on the server (secrets, database access). Some code must run on the client (user interactions). Some code can run either place (rendering). The art of fullstack is knowing which tradeoffs matter for your use case.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Frontend | Code that runs in the user's browser | Any web development discussion |
| Backend | Code that runs on a server you control | API development, data processing |
| Fullstack | Working on both frontend and backend | Job titles, framework descriptions |
| Client | The browser or app making requests (same as frontend in web context) | API documentation, debugging |
| Server | A computer that receives requests and sends responses | Deployment, architecture discussions |
| API | The interface between frontend and backend (usually HTTP + JSON) | Any feature involving data |
| Database | Where data is permanently stored | Any feature involving data |
| Endpoint | A specific URL that the backend responds to (e.g., /api/users) | API design, frontend integration |
| SSR | Server-Side Rendering: generating HTML on the server | Performance, SEO discussions |
| CSR | Client-Side Rendering: generating HTML in the browser | SPA architecture |
| SSG | Static Site Generation: generating HTML at build time | Blogs, marketing sites |
| Hydration | Making server-rendered HTML interactive by attaching JavaScript | React/Next.js debugging |
| Edge | Servers distributed globally, close to users | Performance optimization |
| Serverless | Backend code that runs on-demand, not on an always-on server | AWS Lambda, Vercel Functions |
| CDN | Content Delivery Network: globally distributed cache for static files | Performance, deployment |
| State | Data that changes over time (UI state, server state) | React, frontend architecture |
| Business logic | The rules specific to your application (pricing, permissions, workflows) | Backend development |
| Migration | A controlled change to the database schema | Database management |
| Deploy | Putting code into production where users can access it | Any shipping discussion |

## Common Patterns

### Pattern 1: Static Marketing Site (Frontend Only)

**When to use it:** Landing pages, documentation, blogs, content sites where the content is the same for everyone.

**How it works:** HTML/CSS/JS files are generated at build time and served from a CDN. No backend required.

```
┌─────────────────────────────────────────┐
│           CDN (Vercel, Netlify)         │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │
│  │HTML │ │ CSS │ │ JS  │ │Images│       │
│  └─────┘ └─────┘ └─────┘ └─────┘       │
└─────────────────────────────────────────┘
              ▲
              │ request
           [User]
```

**What you need:**
- A static site generator (Next.js, Astro, Hugo) or plain HTML
- A hosting platform (Vercel, Netlify, GitHub Pages)
- A domain name

**What you don't need:**
- A server
- A database
- Authentication

**Example stack:** Next.js with static export, deployed on Vercel, content in Markdown files.

### Pattern 2: Dashboard App (Frontend + Backend + Database)

**When to use it:** Any app where users have accounts and their own data. SaaS products, admin panels, internal tools.

**How it works:** Frontend handles UI, backend handles logic and data, database stores state.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│   Database   │
│   (React)    │◀────│ (Node/Python)│◀────│ (PostgreSQL) │
└──────────────┘     └──────────────┘     └──────────────┘
     Browser              Server             Server
```

**The request flow:**
1. User logs in → frontend sends credentials to `/api/login`
2. Backend validates credentials against database
3. Backend returns auth token
4. Frontend stores token, includes it in subsequent requests
5. User views dashboard → frontend requests `/api/dashboard`
6. Backend checks auth, queries database, returns data
7. Frontend renders data

**Example stack:** Next.js (frontend + API routes), Prisma (ORM), PostgreSQL (database), Vercel + Supabase (hosting).

### Pattern 3: Mobile App with Shared Backend

**When to use it:** When you have a web app and mobile apps that share the same data and logic.

**How it works:** The backend exposes an API that multiple clients consume.

```
┌──────────────┐
│   Web App    │─────┐
│   (React)    │     │
└──────────────┘     │     ┌──────────────┐     ┌──────────────┐
                     ├────▶│   Backend    │────▶│   Database   │
┌──────────────┐     │     │    (API)     │◀────│              │
│   iOS App    │─────┤     └──────────────┘     └──────────────┘
│   (Swift)    │     │
└──────────────┘     │
                     │
┌──────────────┐     │
│ Android App  │─────┘
│   (Kotlin)   │
└──────────────┘
```

**Why this pattern:**
- Business logic is centralized (one source of truth)
- Data stays consistent across platforms
- Backend team and mobile teams can work independently
- Features can roll out to web first, then mobile (or vice versa)

**Example stack:** REST or GraphQL API (Node.js/Python/Go), React web app, React Native or Swift/Kotlin mobile apps.

### Pattern 4: Serverless Functions as Lightweight Backend

**When to use it:** Simple backend needs without managing servers. Contact forms, webhooks, scheduled tasks, API proxies.

**How it works:** Individual functions that run on-demand, triggered by HTTP requests or events.

```
┌──────────────┐     ┌──────────────────────────────┐
│   Frontend   │────▶│   Serverless Function        │
│              │◀────│   (AWS Lambda / Vercel)      │
└──────────────┘     │                              │
                     │   exports.handler = async    │
                     │     (event) => {             │
                     │       // process request     │
                     │       return response;       │
                     │     }                        │
                     └──────────────────────────────┘
```

**When serverless works well:**
- Low traffic or unpredictable traffic (pay per use)
- Simple request-response patterns
- No need for persistent connections (WebSockets harder)
- Stateless operations (no in-memory data between requests)

**When serverless struggles:**
- High traffic (traditional servers can be cheaper)
- Long-running operations (timeouts, typically 10-60 seconds)
- Complex applications (many functions become hard to manage)

**Example stack:** Next.js API routes (which are serverless on Vercel), or standalone Cloudflare Workers.

### Pattern 5: BFF (Backend for Frontend)

**When to use it:** When your frontend needs data shaped differently than your core backend provides, or when you have multiple frontends with different needs.

**How it works:** A thin backend layer sits between your frontend and your core services, aggregating and shaping data.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────────┐
│   Web App    │────▶│   Web BFF    │────▶│                      │
└──────────────┘     └──────────────┘     │   Core Backend       │
                                     ├────▶│   Services           │
┌──────────────┐     ┌──────────────┐     │   (Users, Orders,    │
│  Mobile App  │────▶│  Mobile BFF  │────▶│    Products, etc.)   │
└──────────────┘     └──────────────┘     └──────────────────────┘
```

**Why BFF:**
- Web dashboard needs one API call that combines user + orders + recommendations
- Mobile app needs a slimmer payload (less data over cellular)
- Different auth patterns per client
- Frontend teams can own their BFF without touching core services

**Example:** Your React app has a Next.js API route layer that calls your microservices and returns exactly what the components need, in one request instead of five.

## Mistakes Everyone Makes

### 1. Thinking "just move it to the frontend" is always simple

**What people do wrong:** They assume that if something currently runs on the backend, moving it to the frontend is a quick win.

**Why it seems right:** "No backend change needed! We'll just do it in JavaScript!"

**What actually happens:**
- **Security:** Business logic in the frontend can be bypassed. If the backend doesn't re-validate, users can cheat.
- **Data access:** The frontend can only use data the backend sends it. If the calculation needs data the frontend doesn't have, the backend needs to send more (potentially sensitive) data.
- **Consistency:** If the logic runs in the frontend, different browsers might compute different results. Backend logic is centralized.

**Example:** Calculating a discount. If the frontend computes "20% off" and sends the discounted price to the backend, a user can modify the frontend to send "99% off." The backend must always compute prices itself.

**What to do instead:** Ask "who benefits if this runs on the client?" If it's user experience (faster interactions), frontend is fine. If it's about correctness or security, it must be backend.

### 2. Putting business logic in the frontend

**What people do wrong:** They implement important rules (pricing, permissions, validation) in frontend code.

**Why it seems right:** "The frontend needs this logic to show the right thing anyway."

**What actually happens:** Anyone can open DevTools, read your code, and understand your business rules. Worse, they can modify the code or send requests directly to your API, bypassing your frontend logic entirely.

```javascript
// This frontend code is visible to everyone
if (user.subscriptionTier === 'premium') {
  // Show premium features
}

// A user can just call your API directly:
// fetch('/api/premium-feature', { ... })
// The backend MUST check the subscription, not trust the frontend
```

**What to do instead:** Implement business logic on the backend. The frontend can display the result of that logic, but the backend must be the source of truth. If the frontend says "user is premium," the backend should verify it before granting access.

### 3. Assuming frontend changes are faster than backend changes

**What people do wrong:** They estimate "2 days for backend, 1 day for frontend" based on the assumption that UI work is simpler.

**Why it seems right:** "It's just moving pixels around."

**What actually happens:**
- Complex state management can take longer than CRUD endpoints
- Cross-browser testing reveals edge cases
- Accessibility requirements add work
- Animation and polish take time
- The "simple" design has five states: loading, error, empty, partial, complete

**What to do instead:** Estimate based on complexity, not which side of the stack. A new API endpoint that reads from a single table might be 2 hours. A UI component with drag-and-drop, optimistic updates, and error handling might be 2 days.

### 4. Not understanding that a "simple UI change" might need a new API endpoint

**What people do wrong:** They see a design mockup and think "we'll just add this field to the screen."

**Why it seems right:** The field exists in the database. How hard could it be?

**What actually happens:**
1. The API doesn't return that field (security: it wasn't needed before)
2. Backend work: add field to API response (and consider: should everyone see this?)
3. Maybe the field isn't even in the database yet
4. Database work: add column, migrate existing data
5. Now the frontend can actually use it

**The chain:**
```
"Show user's join date on profile"
    │
    ├── Is it in the database? (yes)
    ├── Is it in the API response? (no → backend change)
    ├── Should it be visible to everyone? (no → backend authorization)
    └── Now frontend can display it
```

**What to do instead:** Before estimating UI work, check: does the API already provide this data? If not, the work includes backend changes.

### 5. Confusing server-side rendering with backend

**What people do wrong:** They think "server-side rendering" means "backend development" and conflate the two.

**Why it seems right:** Both happen on a server.

**What actually happens:**

**Server-Side Rendering (SSR):**
- Generates HTML on the server
- Happens at request time (or build time for SSG)
- Goal: faster initial page load, SEO
- Doesn't require a "backend" in the traditional sense

**Backend:**
- Handles business logic, data, authentication
- Exposes APIs for clients to consume
- Persists data to databases
- Enforces security and business rules

You can have SSR without a backend: a static blog with pre-rendered pages. You can have a backend without SSR: a JSON API consumed by a client-rendered React app.

**The confusion:**
```
"We need SSR for SEO"     → Talking about how HTML is generated
"We need a backend"       → Talking about data and logic
"We're doing fullstack"   → Could mean either or both
```

**What to do instead:** Be specific. "We need the page to load with content visible (SSR)" is different from "we need to store and retrieve user data (backend)."

### 6. Treating the frontend as a dumb display layer

**What people do wrong:** They think the frontend "just shows what the backend sends" and don't account for frontend complexity.

**Why it seems right:** The backend is where the "real" logic lives.

**What actually happens:** Modern frontends have significant complexity:
- State management across components
- Optimistic updates (show success before backend confirms)
- Offline support and sync
- Complex form validation with good UX
- Accessibility across devices and assistive technologies
- Performance optimization (code splitting, lazy loading)
- Real-time updates (WebSockets, SSE)

A "simple CRUD screen" might involve: loading states, error states, empty states, pagination, sorting, filtering, inline editing, bulk actions, undo/redo, and keyboard shortcuts.

**What to do instead:** Respect frontend complexity. A skilled frontend developer isn't "just making things pretty." They're building the interface between humans and your system.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Never trust the frontend for security or business logic.** The backend must validate, authorize, and compute anything that matters. The frontend is for UX, not enforcement.

2. **Design the API contract before building either side.** What endpoints exist? What data do they accept and return? Frontend and backend can then build in parallel.

3. **Keep secrets on the server.** API keys, database credentials, encryption keys. If it's in frontend code, assume users can see it.

4. **The database is only accessible from the backend.** This isn't a suggestion. Direct frontend-to-database connections are a security incident waiting to happen.

5. **Match rendering strategy to content.** Static content → SSG. Dynamic but public content → SSR. Personalized or interactive content → CSR. Most apps use a mix.

6. **When in doubt, start with a fullstack framework.** Next.js, Remix, or similar. You can always split later. Starting separated adds complexity you might not need.

7. **API responses should return exactly what the frontend needs, no more.** Don't send all 50 fields when the page only displays 5. Don't send user passwords to the frontend "just in case."

8. **Understand the request lifecycle before debugging.** User action → Frontend code → HTTP request → Backend code → Database query → Response → Frontend code → UI update. Which step failed?

## The "Just Tell Me What to Do" Quickstart

Build a simple fullstack app that displays data from a database. You'll see the complete flow: frontend → API → database → API → frontend.

**Prerequisites:** Node.js installed, basic familiarity with React.

**Step 1: Create a Next.js app**

```bash
npx create-next-app@latest my-fullstack-app
cd my-fullstack-app
```

Choose: TypeScript (yes), ESLint (yes), Tailwind (yes), App Router (yes).

**Step 2: Create a "database" (JSON file for simplicity)**

Create `data/users.json`:

```json
[
  {"id": 1, "name": "Alice", "email": "alice@example.com", "role": "admin"},
  {"id": 2, "name": "Bob", "email": "bob@example.com", "role": "user"},
  {"id": 3, "name": "Charlie", "email": "charlie@example.com", "role": "user"}
]
```

**Step 3: Create the backend (API route)**

Create `app/api/users/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import fs from 'fs';
import path from 'path';

// This code runs on the SERVER
export async function GET() {
  // Read from our "database"
  const filePath = path.join(process.cwd(), 'data', 'users.json');
  const fileContents = fs.readFileSync(filePath, 'utf8');
  const users = JSON.parse(fileContents);

  // In a real app: check authentication, filter by permissions
  // For now, return all users

  console.log('API: Fetching users from database'); // This logs on the server

  return NextResponse.json(users);
}
```

**Step 4: Create the frontend (React component)**

Replace `app/page.tsx`:

```tsx
'use client';

import { useState, useEffect } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

export default function Home() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // This code runs in the BROWSER
    console.log('Frontend: Fetching from API'); // This logs in browser console

    fetch('/api/users')
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        setUsers(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <div className="p-8">Loading...</div>;
  if (error) return <div className="p-8 text-red-500">Error: {error}</div>;

  return (
    <main className="p-8">
      <h1 className="text-2xl font-bold mb-4">Users</h1>
      <div className="border rounded-lg overflow-hidden">
        <table className="w-full">
          <thead className="bg-gray-100">
            <tr>
              <th className="p-3 text-left">Name</th>
              <th className="p-3 text-left">Email</th>
              <th className="p-3 text-left">Role</th>
            </tr>
          </thead>
          <tbody>
            {users.map(user => (
              <tr key={user.id} className="border-t">
                <td className="p-3">{user.name}</td>
                <td className="p-3">{user.email}</td>
                <td className="p-3">
                  <span className={`px-2 py-1 rounded text-sm ${
                    user.role === 'admin'
                      ? 'bg-purple-100 text-purple-800'
                      : 'bg-gray-100 text-gray-800'
                  }`}>
                    {user.role}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </main>
  );
}
```

**Step 5: Run it**

```bash
npm run dev
```

Open `http://localhost:3000`. You'll see the users table.

**Step 6: Trace the flow**

1. Open browser DevTools → Console tab. You'll see "Frontend: Fetching from API"
2. Check your terminal (where `npm run dev` is running). You'll see "API: Fetching users from database"
3. DevTools → Network tab. You'll see the request to `/api/users` and the JSON response

**You just built:**
- Frontend: React component in the browser
- API: Next.js route handler on the server
- Database: JSON file (in production, this would be PostgreSQL, etc.)

The frontend has no idea where the data comes from. It just calls the API. The API could be reading from a JSON file, PostgreSQL, MongoDB, or a third-party service. The frontend doesn't care.

**Step 7: Add a POST endpoint (optional)**

To see the full cycle, add the ability to create users.

Update `app/api/users/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import fs from 'fs';
import path from 'path';

const filePath = path.join(process.cwd(), 'data', 'users.json');

function readUsers() {
  const fileContents = fs.readFileSync(filePath, 'utf8');
  return JSON.parse(fileContents);
}

function writeUsers(users: any[]) {
  fs.writeFileSync(filePath, JSON.stringify(users, null, 2));
}

export async function GET() {
  const users = readUsers();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();

  // Validation (backend responsibility!)
  if (!body.name || !body.email) {
    return NextResponse.json(
      { error: 'Name and email are required' },
      { status: 400 }
    );
  }

  const users = readUsers();
  const newUser = {
    id: Math.max(...users.map((u: any) => u.id)) + 1,
    name: body.name,
    email: body.email,
    role: 'user' // Default role, not from user input!
  };

  users.push(newUser);
  writeUsers(users);

  return NextResponse.json(newUser, { status: 201 });
}
```

Notice: the role is set by the backend, not by user input. This is security 101.

## How to Prompt AI Tools About This

### Context to provide

When asking Claude, Cursor, or Copilot about frontend/backend architecture:
- What you're building (dashboard, API, marketing site)
- What framework you're using (Next.js, React + Express, etc.)
- Whether you need authentication
- What data you need to store and display
- Performance requirements (SEO, speed)

### Example prompts that produce good results

**Starting a project:**
> "I'm building a SaaS dashboard with Next.js. Users will have accounts and see their own data. I need user authentication, a PostgreSQL database, and a few CRUD screens. Help me plan the architecture: what's frontend, what's backend, how do they communicate?"

**Understanding boundaries:**
> "In my Next.js app, I have a pricing calculation that's currently in a React component. It uses user subscription tier and product prices. Should this logic be in the frontend or backend? Walk me through the security implications."

**API design:**
> "I need to build an API endpoint that returns a user's dashboard data. The dashboard shows: recent orders, account balance, and notifications. Should this be one endpoint or three? What are the tradeoffs?"

**Rendering strategy:**
> "My marketing site is currently client-rendered React. We need better SEO. Should I switch to SSR or SSG? The site has 50 pages, updated weekly."

### What to watch out for in AI-generated architecture advice

- **Putting secrets in frontend code.** AI might suggest `const API_KEY = 'sk-...'` in a React component. Always move secrets to the backend.
- **Direct database access from frontend.** AI might import a database client in a React component. Only API routes (server code) should touch databases.
- **Missing validation on the backend.** AI might validate form input only on the frontend. Always re-validate on the backend.
- **Unclear about where code runs.** AI might write a function without specifying if it's for the browser or server. Ask: "Where does this code execute?"
- **Over-engineering early.** AI might suggest microservices when a monolith would be simpler. Start with the simplest architecture that works.

### Key terms to use in prompts

Using these terms gets better output: "client-side," "server-side," "API route," "database query," "authentication middleware," "SSR vs CSR," "API contract," "JSON response," "request validation," "environment variables."

## Ship It: Build This

Build a fullstack task manager with Next.js that demonstrates the complete frontend → API → database flow.

**What you're building:** A simple task manager where users can view tasks, add new tasks, and mark them complete. The UI is in React, the API is Next.js route handlers, and the database is SQLite (file-based, no setup required).

**Why this project specifically:** It exercises the full stack with real database operations (not just a JSON file). You'll see CREATE, READ, and UPDATE operations, handle loading and error states, and trace data through all layers.

**Rough architecture:**

```
┌─────────────────────────────────────────────────────┐
│  Frontend (React)                                   │
│  - Task list component                              │
│  - Add task form                                    │
│  - Toggle complete button                           │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  API Routes (Next.js)                               │
│  GET  /api/tasks     → List all tasks              │
│  POST /api/tasks     → Create new task             │
│  PATCH /api/tasks/[id] → Update task (complete)    │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  Database (SQLite via better-sqlite3)              │
│  Table: tasks (id, title, completed, created_at)   │
└─────────────────────────────────────────────────────┘
```

**What you'll learn:**
- How a form submission flows from frontend → API → database
- How to structure API routes for CRUD operations
- How to handle optimistic updates in the UI
- The difference between what runs in the browser vs. on the server

**Deliverables:**
1. A working task manager at `http://localhost:3000`
2. Ability to add tasks, view tasks, and toggle completion
3. Data persists across page refreshes (stored in SQLite)
4. Console logs showing exactly where each piece of code runs

**Estimated time:** 2-3 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn GraphQL** when your frontend needs flexible data fetching and you're tired of creating one-off REST endpoints. Useful when different views need different subsets of the same data.
- **Learn microservices architecture** when your monolith is too big for one team or deployment. But only then. Most apps should stay monolithic longer than you think.
- **Learn WebSockets** when you need real-time communication (chat, live dashboards, collaborative editing). See the [WebSockets guide](../websockets/).
- **Learn edge computing (Cloudflare Workers, Vercel Edge)** when latency matters and you can run logic closer to users. Good for A/B testing, authentication, and simple API responses.
- **Learn database design patterns** when your app's data model is getting complex. Normalization, indexes, and query optimization have a huge impact on performance.
