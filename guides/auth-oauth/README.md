# Authentication and OAuth: Zero to Ship

> Proving who you are to a website, the same way you show your ID at a bar, except the bar forgets your face every time you walk away from the counter.

## Why Should You Care?

Every app with user accounts has authentication. Login pages, "Sign in with Google" buttons, protected dashboards, API keys, session cookies. Auth is everywhere, and it's the one thing you absolutely cannot get wrong. A bug in your todo list feature means someone sees the wrong task. A bug in your auth means someone accesses another person's data, financial information, medical records, private messages.

If you build with AI tools, you're generating auth code constantly. Cursor scaffolds login flows. Claude Code writes middleware to protect routes. v0 generates sign-up pages with OAuth buttons. The code usually looks right. But auth is the domain where "looks right" is most dangerous. A JWT that's not validated on the server, a cookie without the right flags, a token stored in localStorage instead of an httpOnly cookie, these are invisible bugs that work perfectly in development and become security vulnerabilities in production.

The other reason you need to understand auth: it's the number one thing that blocks you when building. "I can't get Google login to work." "My API returns 401 and I don't know why." "The user is logged in on one page but not another." These are all auth problems, and they're all solvable in minutes if you understand how auth actually works. Without that understanding, you're guessing and copying Stack Overflow answers until something sticks.

## The 30-Second Version

Authentication (authn) proves who you are. Authorization (authz) decides what you're allowed to do. Sessions use cookies to remember you. JWTs encode your identity into a token the server can verify without a database lookup. OAuth lets you log in with Google/GitHub/etc. by redirecting you to their login page, getting a temporary code, and exchanging it for user info. For most apps, use an auth library (NextAuth, Clerk, Supabase Auth) instead of building auth yourself. Roll-your-own auth is the number one source of security vulnerabilities in web apps.

## The Real Explanation (ELI5)

Imagine you're going to a music festival that lasts three days.

**Day 1 (authentication):** You walk up to the gate. The guard asks for your ticket and your government ID. You show both. The guard checks that the name on the ticket matches your ID. That's authentication: proving you are who you claim to be.

**The wristband (session/token):** After checking your ID, the guard puts a wristband on your wrist. For the rest of the festival, you just flash your wristband. No one asks for your ID again. The wristband IS your proof that you've been verified. That's a session token (or a JWT). The initial check was expensive (database lookup, password verification). The wristband is cheap to verify (just look at it).

**Different colored wristbands (authorization):** VIP wristbands get you backstage. Regular wristbands keep you in the general area. Both wristbands prove you're a valid attendee (authentication), but they grant different access (authorization). That's the difference between authn and authz.

**OAuth ("Sign in with Google"):** Imagine instead of showing your own ID at the festival gate, you say "I'm on the Google guest list." The guard calls Google: "Is this person legit?" Google says "Yes, here's their name and email." The guard lets you in and gives you a wristband. You never showed the festival your ID or password directly. Google vouched for you. That's OAuth: a trusted third party confirms your identity.

Now replace the festival with a website, the guard with your backend server, the ID check with password hashing, the wristband with a cookie or JWT, and the Google phone call with the OAuth redirect flow. That's web authentication.

The analogy breaks at one key point: at the festival, the guard can see your physical wristband and knows it's real. On the internet, the "wristband" (token) travels over the network, so it must be cryptographically signed to prevent forgery. Anyone can create a string that says "user_id: 42". Only your server can create a string that's signed with your secret key and provably hasn't been tampered with.

## How It Actually Works

### Authentication vs Authorization

These two words sound similar but mean completely different things:

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| **Question** | "Who are you?" | "What are you allowed to do?" |
| **When** | Login, every request with a token | After authentication, when accessing a resource |
| **How** | Password, OAuth, biometrics, API key | Roles, permissions, policies, row-level security |
| **Failure** | 401 Unauthorized ("I don't know who you are") | 403 Forbidden ("I know who you are, but you can't do this") |

**Example flow:**
1. User logs in with email + password → **Authentication** (proves identity)
2. Server issues a session/token → **Authentication** (remembers identity)
3. User tries to access `/admin/dashboard` → **Authorization** (checks if user has admin role)
4. User tries to view another user's profile → **Authorization** (checks if this is allowed)

In most apps, authentication happens once (at login) and the result is carried forward in a token. Authorization happens on every request when the server checks what the authenticated user is allowed to do.

### Sessions: The Cookie Approach

Sessions are the oldest and most battle-tested approach to web authentication.

```
1. User submits email + password
2. Server verifies password against stored hash
3. Server creates a session record in a database:
   { session_id: "abc123", user_id: 42, expires_at: "..." }
4. Server sends a cookie to the browser:
   Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
5. Browser automatically sends this cookie with every subsequent request
6. Server reads the cookie, looks up the session in the database, knows who the user is
```

```
Browser                          Server                          Database
  │                                │                                │
  │  POST /login                   │                                │
  │  { email, password }           │                                │
  │───────────────────────────────▶│                                │
  │                                │  Look up user by email         │
  │                                │───────────────────────────────▶│
  │                                │  User { id: 42, hash: "$2b..."} │
  │                                │◀───────────────────────────────│
  │                                │                                │
  │                                │  Verify password against hash  │
  │                                │  ✓ Match                       │
  │                                │                                │
  │                                │  Store session                 │
  │                                │───────────────────────────────▶│
  │                                │  { sid: "abc", user: 42 }     │
  │                                │◀───────────────────────────────│
  │                                │                                │
  │  Set-Cookie: session_id=abc    │                                │
  │◀───────────────────────────────│                                │
  │                                │                                │
  │  GET /dashboard                │                                │
  │  Cookie: session_id=abc        │                                │
  │───────────────────────────────▶│                                │
  │                                │  Look up session "abc"         │
  │                                │───────────────────────────────▶│
  │                                │  { user_id: 42, expires: ... } │
  │                                │◀───────────────────────────────│
  │                                │                                │
  │  200 OK (dashboard data)       │                                │
  │◀───────────────────────────────│                                │
```

**Cookie flags that matter:**

| Flag | What it does | Why it matters |
|------|-------------|----------------|
| `HttpOnly` | JavaScript can't read the cookie | Prevents XSS attacks from stealing the session |
| `Secure` | Cookie only sent over HTTPS | Prevents interception on insecure networks |
| `SameSite=Lax` | Cookie only sent for same-site requests (with safe exceptions for navigation) | Prevents CSRF attacks |
| `Path=/` | Cookie sent for all paths | Ensures the session works across your whole app |
| `Max-Age=86400` | Cookie expires after 24 hours | Sessions shouldn't last forever |

**Advantages of sessions:**
- Simple to understand and implement.
- Easy to revoke (delete the session from the database, user is immediately logged out).
- The server is in full control. Token can't be forged or manipulated.

**Disadvantages:**
- Every request requires a database lookup to validate the session.
- Harder to scale across multiple servers (need shared session store like Redis).
- Cookies don't work well for mobile apps or third-party API access.

### JWTs: The Token Approach

A JWT (JSON Web Token) is a self-contained token. Instead of storing the session in a database and giving the browser a session ID, you encode the user's identity directly into the token. The server can verify the token without any database lookup.

**What's inside a JWT:**

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0Miwicm9sZSI6ImFkbWluIiwiZXhwIjoxNzEwMDAwMDAwfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

This is three parts separated by dots:

HEADER.PAYLOAD.SIGNATURE

Header (base64 decoded):
{
  "alg": "HS256",        ← signing algorithm
  "typ": "JWT"
}

Payload (base64 decoded):
{
  "user_id": 42,          ← who this token represents
  "role": "admin",        ← their role (authorization info)
  "exp": 1710000000,      ← when this token expires (Unix timestamp)
  "iat": 1709913600       ← when this token was issued
}

Signature:
HMAC-SHA256(
  base64(header) + "." + base64(payload),
  "your-secret-key"       ← only the server knows this
)
```

**Important:** The payload is base64-encoded, NOT encrypted. Anyone can decode it and read the contents. The signature only prevents tampering. If someone changes `"role": "admin"` to `"role": "superadmin"`, the signature won't match and the server rejects it. But anyone can read that the user_id is 42 and the role is admin. Never put sensitive data (passwords, credit card numbers, secrets) in a JWT.

**JWT flow:**

```
1. User logs in with email + password
2. Server verifies password
3. Server creates a JWT containing user_id, role, expiration
4. Server signs the JWT with a secret key
5. Server sends the JWT to the client
6. Client stores it (cookie or memory) and sends it with every request
7. Server verifies the signature and reads the payload — no database needed
```

**Advantages of JWTs:**
- No database lookup per request (the token contains all needed info).
- Works great for APIs, mobile apps, and cross-domain scenarios.
- Scales easily across multiple servers (any server with the secret key can verify).

**Disadvantages:**
- Can't be revoked easily. Once issued, a JWT is valid until it expires. If a user's account is compromised, you can't "delete" their JWT like you can delete a session. You have to wait for expiration or maintain a revocation list (which brings back the database lookup problem).
- Token size. JWTs are larger than session IDs (hundreds of bytes vs ~32 bytes).
- Complexity. Getting JWT storage, refresh, and validation right has many subtle pitfalls.

### OAuth 2.0: Letting Someone Else Handle Login

OAuth 2.0 is a protocol that lets users log into your app using their existing account on another service (Google, GitHub, Apple, etc.). The user never gives your app their Google password. Instead, Google confirms the user's identity and gives your app limited information about them.

**The real-world scenario:**

You're building a web app. You want users to sign up. You could build email/password registration (password hashing, email verification, forgot password flow, account lockout). That's a lot of work and a lot of security surface area. Or you could add a "Sign in with Google" button. Google handles the login, verifies the identity, and tells your app "yes, this is priya@gmail.com."

**The Authorization Code Flow (step by step):**

This is the most common and most secure OAuth flow for web apps.

```
User          Your App (Frontend)      Your App (Backend)      Google
 │                  │                        │                    │
 │ Click "Sign in   │                        │                    │
 │  with Google"    │                        │                    │
 │─────────────────▶│                        │                    │
 │                  │                        │                    │
 │   Redirect to Google's login page         │                    │
 │◀─────────────────│                        │                    │
 │   URL: https://accounts.google.com/o/oauth2/auth              │
 │   ?client_id=YOUR_CLIENT_ID                                   │
 │   &redirect_uri=https://yourapp.com/auth/callback              │
 │   &response_type=code                                         │
 │   &scope=openid email profile                                 │
 │   &state=random_csrf_token                                    │
 │                  │                        │                    │
 │ User logs in     │                        │                    │
 │ at Google        │                        │                    │
 │──────────────────────────────────────────────────────────────▶│
 │                  │                        │                    │
 │ "Allow YourApp   │                        │                    │
 │  to see your     │                        │                    │
 │  email?"  → Yes  │                        │                    │
 │──────────────────────────────────────────────────────────────▶│
 │                  │                        │                    │
 │   Redirect back to your app with a temporary code             │
 │◀──────────────────────────────────────────────────────────────│
 │   URL: https://yourapp.com/auth/callback                      │
 │   ?code=TEMPORARY_AUTH_CODE                                   │
 │   &state=random_csrf_token                                    │
 │                  │                        │                    │
 │                  │  Forward code to       │                    │
 │                  │  backend               │                    │
 │                  │───────────────────────▶│                    │
 │                  │                        │                    │
 │                  │                        │  Exchange code     │
 │                  │                        │  for access token  │
 │                  │                        │  (server-to-server)│
 │                  │                        │──────────────────▶│
 │                  │                        │                    │
 │                  │                        │  Access token +    │
 │                  │                        │  user info         │
 │                  │                        │◀──────────────────│
 │                  │                        │                    │
 │                  │  Create session/JWT    │                    │
 │                  │  for your app          │                    │
 │                  │◀───────────────────────│                    │
 │                  │                        │                    │
 │  Logged in!      │                        │                    │
 │◀─────────────────│                        │                    │
```

**Why is it this complicated?** Security. The temporary code is exchanged for the access token on the server side (step where your backend talks to Google directly). This prevents the access token from ever appearing in the browser's URL bar, JavaScript, or network logs. If the code is intercepted, it's useless without your app's client secret (which only your server knows).

**Key terms in the OAuth flow:**
- **Client ID:** Your app's public identifier (safe to expose in frontend code).
- **Client Secret:** Your app's private key (NEVER expose in frontend code, server-side only).
- **Authorization Code:** The temporary code Google gives you after the user approves. Short-lived, one-time use.
- **Access Token:** The token you exchange the code for. Lets you call Google APIs on behalf of the user.
- **Redirect URI:** Where Google sends the user back after login. Must match exactly what you registered.
- **Scope:** What information/access you're requesting (email, profile, calendar, etc.).
- **State:** A random string you generate to prevent CSRF attacks. You send it, Google sends it back, you verify it matches.

### Social Login Under the Hood

"Sign in with Google" uses OAuth 2.0 + OpenID Connect (OIDC). OpenID Connect is a thin layer on top of OAuth that standardizes how you get user identity information.

Here's what actually happens when you click that button:

1. Your app redirects to `accounts.google.com` with your client_id and requested scopes.
2. Google shows the user a login screen (if not already logged in) and a consent screen ("Allow YourApp to see your email and profile?").
3. User approves. Google redirects back to your `redirect_uri` with a temporary `code`.
4. Your backend exchanges the `code` (plus your `client_secret`) for an `access_token` and an `id_token`.
5. The `id_token` is a JWT containing the user's info (email, name, profile picture, Google user ID).
6. Your backend verifies the `id_token` signature, extracts the user info.
7. Your backend creates or finds a user record in your database ("upsert").
8. Your backend creates a session or JWT for YOUR app and sends it to the browser.

From this point on, your app's own session/JWT handles everything. The Google tokens are only used during the login flow.

### API Key Auth vs Token Auth

| | API Key | Bearer Token (JWT/Session) |
|---|---|---|
| **Identifies** | The application | The user |
| **Issued to** | A developer/team | An individual user after login |
| **Lifetime** | Long-lived (months/years) | Short-lived (minutes/hours/days) |
| **Revocation** | Regenerate the key | Delete session or wait for JWT expiry |
| **Use case** | Server-to-server calls, third-party APIs | User-facing apps, per-user access |
| **Example** | `Authorization: Bearer sk-abc123` or `X-API-Key: abc123` | `Authorization: Bearer eyJhbGci...` |

**When to use API keys:**
- Your backend calling another service (Stripe, OpenAI, Supabase).
- Third-party developers accessing your API.
- Service-to-service communication where there's no "user."
- Webhook verification.

**When to use tokens:**
- User login on a web or mobile app.
- Anything where you need to know WHICH user is making the request.
- Anything where permissions differ per user.

### Password Hashing

Never store passwords in plain text. Not "mostly never." Never.

**What bcrypt does:**

```
User password: "mypassword123"
         ↓
bcrypt("mypassword123", salt_rounds=10)
         ↓
Stored in database: "$2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
```

The stored hash is a one-way transformation. You can go from password to hash, but you can't go from hash to password. When the user logs in again, you hash the submitted password with the same algorithm and compare the hashes.

**Why bcrypt specifically:**
- It's intentionally slow (configurable via "salt rounds"). This makes brute-force attacks take years instead of seconds.
- Each hash includes a random salt, so two users with the same password get different hashes. An attacker can't use a precomputed lookup table.
- It's battle-tested for decades.

```python
# Python
import bcrypt

# Hash a password (during registration)
password = "mypassword123".encode("utf-8")
hashed = bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))
# Store `hashed` in the database

# Verify a password (during login)
submitted = "mypassword123".encode("utf-8")
if bcrypt.checkpw(submitted, hashed):
    print("Password correct")
else:
    print("Wrong password")
```

```javascript
// Node.js
import bcrypt from "bcryptjs";

// Hash
const hash = await bcrypt.hash("mypassword123", 12);
// Store `hash` in the database

// Verify
const match = await bcrypt.compare("mypassword123", hash);
```

**Salt rounds:** The number controls how slow the hashing is. 10-12 is standard. Each increase doubles the time. At 10 rounds, hashing takes ~100ms. At 12, ~300ms. This is fast enough for login but slow enough that an attacker trying billions of passwords would take centuries.

### Auth Libraries: Don't Build It Yourself

Auth is the one area where "let me build it from scratch to learn" is genuinely dangerous in production. Use a library.

| Library | Best for | How it works |
|---------|----------|-------------|
| **NextAuth / Auth.js** | Next.js apps | Runs in your app. Handles OAuth, sessions, JWT, database adapters. Free, open source. You control the data. |
| **Clerk** | Fast setup, managed | Hosted auth service. Drop-in UI components. Handles everything including user management UI. Paid beyond free tier. |
| **Supabase Auth** | Supabase projects | Built into Supabase. Row-level security integration. OAuth + email/password + magic links. Free with Supabase. |
| **Firebase Auth** | Firebase/Google Cloud projects | Similar to Supabase Auth. Great mobile support. Tight integration with Firebase services. |
| **Lucia** | Full control + security | Minimal, bring-your-own-database auth library. No magic, no opinions. You write the login UI and flows. For developers who want to understand every line. |

**When to build your own (rare):**
- You're learning and the app will never see production.
- You have extremely specific auth requirements that no library supports.
- You're building the auth library itself.

**When to use a library (almost always):**
- You're building a product that real people will use.
- You need OAuth (the redirect flows are easy to get subtly wrong).
- You want password reset, email verification, rate limiting on login, account lockout, and all the other things that "simple auth" eventually needs.

## The Mental Model

**Mental model 1: "Authentication is the bouncer, authorization is the VIP list."** The bouncer checks your ID (authentication). The VIP list determines which rooms you can enter (authorization). These are two separate checks. You can pass authentication (valid user) and still fail authorization (not an admin). Always implement them as separate concerns in your code.

**Mental model 2: "Sessions are like coat checks, JWTs are like wristbands."** With a session (coat check), you hand over your identity proof and get a ticket (session ID). To verify you, the server looks up the ticket in the database (checks the coat room). With a JWT (wristband), your identity is embedded in the token itself, signed by the server. Anyone with the server's public key can verify it without calling the database. Sessions are easier to revoke (destroy the ticket). JWTs are faster to verify (no database call). Choose based on your tradeoffs.

**Mental model 3: "OAuth is introducing a friend through a mutual contact."** You don't give the app your Google password. Instead, Google introduces you: "I've verified this person. Here's their name and email." Your app trusts Google's introduction. This is why OAuth is called "delegated authorization": you're delegating the identity check to a party both sides trust.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Authentication (authn) | Proving who you are (login) | Every app with users |
| Authorization (authz) | Deciding what you're allowed to do (permissions) | Admin panels, role-based features |
| Session | Server-side record that remembers a logged-in user, identified by a cookie | Traditional web apps |
| Cookie | A small piece of data the browser stores and sends with every request to a domain | Session management, preferences |
| JWT (JSON Web Token) | A self-contained token with user info, signed by the server | APIs, SPAs, mobile apps |
| OAuth 2.0 | A protocol for logging in via a third-party service (Google, GitHub) | "Sign in with" buttons |
| OpenID Connect (OIDC) | A layer on top of OAuth that standardizes getting user identity info | Social login implementations |
| Authorization Code | A temporary one-time code exchanged for an access token during OAuth | OAuth redirect flow |
| Access Token | A token that lets you call APIs on behalf of a user | After OAuth login |
| Refresh Token | A long-lived token used to get a new access token without re-logging in | Token-based auth systems |
| Client ID | Your app's public identifier registered with an OAuth provider | OAuth configuration |
| Client Secret | Your app's private key for OAuth (server-side only) | OAuth backend configuration |
| Redirect URI | The URL the OAuth provider sends the user back to after login | OAuth configuration |
| Scope | What permissions you're requesting from the OAuth provider | OAuth configuration |
| bcrypt | A password hashing algorithm designed to be intentionally slow | Storing passwords safely |
| Salt | Random data added to a password before hashing to prevent precomputed attacks | Password hashing |
| CSRF | Cross-Site Request Forgery: tricking a user's browser into making unwanted requests | Security, cookie-based auth |
| XSS | Cross-Site Scripting: injecting malicious scripts into a web page | Security, token storage |
| CORS | Cross-Origin Resource Sharing: browser security that controls cross-domain requests | API configuration |
| RBAC | Role-Based Access Control: permissions based on user roles (admin, editor, viewer) | Authorization systems |
| PKCE | Proof Key for Code Exchange: an OAuth extension for public clients (SPAs, mobile) | OAuth in frontend-only apps |
| httpOnly cookie | A cookie that JavaScript can't read (only the server can) | Secure token storage |
| 401 Unauthorized | HTTP status: "I don't know who you are (authenticate first)" | Missing or invalid token |
| 403 Forbidden | HTTP status: "I know who you are, but you can't do this" | Insufficient permissions |

## Common Patterns

### 1. Session-Based Web App Auth

**When to use it:** Server-rendered web apps or any app where the backend and frontend share the same domain. The most straightforward approach.

**How it works:** Server creates a session on login, stores it in a database (or Redis), sends a session ID cookie to the browser. Browser sends the cookie automatically. Server looks up the session on every request.

**Code sketch:**

```python
# FastAPI with session-based auth
from fastapi import FastAPI, Response, Request, HTTPException
from uuid import uuid4
import bcrypt

app = FastAPI()
sessions = {}  # In production, use Redis

@app.post("/login")
async def login(email: str, password: str, response: Response):
    user = await db.get_user_by_email(email)
    if not user or not bcrypt.checkpw(password.encode(), user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid credentials")

    session_id = str(uuid4())
    sessions[session_id] = {"user_id": user.id, "role": user.role}

    response.set_cookie(
        key="session_id",
        value=session_id,
        httponly=True,     # JavaScript can't read it
        secure=True,       # HTTPS only
        samesite="lax",    # CSRF protection
        max_age=86400,     # 24 hours
    )
    return {"message": "Logged in"}

@app.get("/me")
async def get_me(request: Request):
    session_id = request.cookies.get("session_id")
    if not session_id or session_id not in sessions:
        raise HTTPException(status_code=401)

    session = sessions[session_id]
    user = await db.get_user(session["user_id"])
    return {"id": user.id, "name": user.name, "role": session["role"]}

@app.post("/logout")
async def logout(request: Request, response: Response):
    session_id = request.cookies.get("session_id")
    if session_id:
        sessions.pop(session_id, None)
    response.delete_cookie("session_id")
    return {"message": "Logged out"}
```

### 2. JWT API Auth

**When to use it:** APIs consumed by mobile apps, SPAs, or third-party clients. When you need stateless authentication that works across services.

**How it works:** Server issues a signed JWT on login. Client stores it and sends it in the Authorization header. Server verifies the signature and reads the payload. No database lookup needed for verification.

**Code sketch:**

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"  # From environment variable in production

def create_token(user_id: int, role: str) -> str:
    payload = {
        "user_id": user_id,
        "role": role,
        "exp": datetime.utcnow() + timedelta(hours=1),
        "iat": datetime.utcnow(),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm="HS256")

def verify_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")

# Middleware / dependency
async def get_current_user(request: Request):
    auth_header = request.headers.get("Authorization")
    if not auth_header or not auth_header.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Missing token")

    token = auth_header.split(" ")[1]
    payload = verify_token(token)
    return payload  # {"user_id": 42, "role": "admin", ...}

@app.post("/login")
async def login(email: str, password: str):
    user = await db.get_user_by_email(email)
    if not user or not bcrypt.checkpw(password.encode(), user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid credentials")

    access_token = create_token(user.id, user.role)
    refresh_token = create_token(user.id, user.role)  # longer expiry

    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in": 3600,
    }

@app.get("/protected")
async def protected_route(user=Depends(get_current_user)):
    return {"message": f"Hello user {user['user_id']}"}
```

### 3. OAuth Social Login

**When to use it:** You want "Sign in with Google/GitHub/etc." This is the most common pattern in modern web apps.

**How it works:** Redirect to the provider, get a code back, exchange it for user info on the server, create or find the user in your database, start a session.

**Code sketch (Next.js with Auth.js v5):**

```typescript
// auth.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async session({ session, token }) {
      // Add user ID to session
      session.user.id = token.sub!
      return session
    },
  },
})

// app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"
export const { GET, POST } = handlers

// app/page.tsx (using the session)
import { auth } from "@/auth"

export default async function Home() {
  const session = await auth()

  if (!session) {
    return <a href="/api/auth/signin">Sign in</a>
  }

  return <p>Hello, {session.user.name}</p>
}
```

### 4. API Key for Service-to-Service

**When to use it:** Backend services calling each other, webhook verification, third-party developer access to your API.

**How it works:** Generate a random key, store its hash in the database, give the key to the client. The client sends it with every request. Your server hashes the received key and checks it against the database.

**Code sketch:**

```python
import secrets
import hashlib

def generate_api_key() -> tuple[str, str]:
    """Returns (raw_key_for_client, hash_for_database)."""
    raw_key = f"sk_{secrets.token_urlsafe(32)}"
    key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
    return raw_key, key_hash

def verify_api_key(provided_key: str) -> dict | None:
    """Look up the API key in the database."""
    key_hash = hashlib.sha256(provided_key.encode()).hexdigest()
    record = db.query("SELECT * FROM api_keys WHERE key_hash = $1", key_hash)
    if not record or record.revoked:
        return None
    return {"team_id": record.team_id, "scopes": record.scopes}

# Middleware
async def require_api_key(request: Request):
    key = request.headers.get("X-API-Key") or request.headers.get("Authorization", "").replace("Bearer ", "")
    if not key:
        raise HTTPException(status_code=401, detail="API key required")

    identity = verify_api_key(key)
    if not identity:
        raise HTTPException(status_code=401, detail="Invalid API key")

    return identity
```

### 5. Role-Based Access Control (RBAC)

**When to use it:** Different users need different levels of access (admin, editor, viewer). Most apps need this eventually.

**How it works:** Each user has a role. Each route or action checks the user's role before proceeding.

**Code sketch:**

```python
from enum import Enum
from functools import wraps

class Role(str, Enum):
    VIEWER = "viewer"
    EDITOR = "editor"
    ADMIN = "admin"

# Role hierarchy: admin > editor > viewer
ROLE_HIERARCHY = {Role.VIEWER: 0, Role.EDITOR: 1, Role.ADMIN: 2}

def require_role(minimum_role: Role):
    """Decorator that checks if the user has sufficient role."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, user=Depends(get_current_user), **kwargs):
            user_level = ROLE_HIERARCHY.get(user["role"], 0)
            required_level = ROLE_HIERARCHY[minimum_role]
            if user_level < required_level:
                raise HTTPException(
                    status_code=403,
                    detail=f"Requires {minimum_role.value} role"
                )
            return await func(*args, user=user, **kwargs)
        return wrapper
    return decorator

@app.get("/admin/users")
@require_role(Role.ADMIN)
async def list_all_users(user: dict):
    return await db.get_all_users()

@app.put("/posts/{post_id}")
@require_role(Role.EDITOR)
async def update_post(post_id: int, user: dict):
    return await db.update_post(post_id, ...)

@app.get("/posts")
@require_role(Role.VIEWER)
async def list_posts(user: dict):
    return await db.get_posts()
```

For more complex permissions (user A can edit their own posts but not user B's), you need resource-level authorization, not just role checks. Check both the role AND whether the resource belongs to the user.

## Mistakes Everyone Makes

### 1. Storing JWTs in localStorage (XSS risk)

**What people do:** `localStorage.setItem("token", jwt)` and read it on every request.

**Why it seems right:** "It's simple and persists across page refreshes."

**What actually happens:** Any JavaScript running on your page can read localStorage. If your app has a single XSS vulnerability (and most apps eventually do), the attacker's script can steal the JWT, send it to their server, and impersonate the user from anywhere. The token works until it expires. Unlike a session, you can't revoke it.

**What to do instead:** Store JWTs in httpOnly cookies. JavaScript can't read httpOnly cookies, so XSS attacks can't steal the token. The browser sends the cookie automatically with every request. If you must use JWTs with an Authorization header (for API calls from a SPA), store the token in memory (a JavaScript variable), not localStorage or sessionStorage. Yes, the user has to re-login on page refresh. That's the tradeoff for security. Or use a short-lived access token in memory + a long-lived refresh token in an httpOnly cookie.

### 2. Not validating JWT signatures

**What people do:** Decode the JWT payload (base64) and trust its contents without verifying the signature.

**Why it seems right:** "I just need the user_id from the token."

**What actually happens:** Anyone can create a JWT with any payload. Without signature verification, an attacker creates `{"user_id": 1, "role": "admin"}`, base64-encodes it, and your server treats them as admin. This is not a theoretical attack. It happens when developers use `jwt.decode()` without passing the secret key or set `algorithms` to include `none`.

**What to do instead:** Always verify the signature with your secret key. Always specify the allowed algorithm explicitly (never accept `alg: none`). Use your JWT library's `verify` function, not just `decode`.

```python
# BAD: just decodes, no verification
payload = jwt.decode(token, options={"verify_signature": False})

# GOOD: verifies signature with secret key and specific algorithm
payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
```

### 3. Building custom auth instead of using a library

**What people do:** Write their own login flow, password hashing, session management, OAuth integration, password reset, and email verification from scratch.

**Why it seems right:** "Auth is simple. It's just a POST endpoint that checks a password and returns a token."

**What actually happens:** Simple auth works for a demo. Then you need: password reset emails, email verification, rate limiting on login attempts, account lockout after failed attempts, CSRF protection, secure cookie configuration, token refresh, session invalidation on password change, OAuth state parameter validation, PKCE for mobile clients, and handling edge cases like "user signed up with email, then tries Google OAuth with the same email." Each of these is a subtle security requirement that's easy to get wrong. Auth libraries have been battle-tested by thousands of developers and security researchers. Your weekend auth code has not.

**What to do instead:** Use NextAuth/Auth.js for Next.js, Clerk for fast setup, Supabase Auth if you're on Supabase, or Lucia if you want low-level control with safety. All of these handle the hard parts.

### 4. Forgetting to handle token refresh

**What people do:** Issue a JWT with a 24-hour expiry and don't implement refresh.

**Why it seems right:** "24 hours is long enough."

**What actually happens:** The user is working in your app. 24 hours after login, their token expires. The next API call returns 401. The user sees an error or gets kicked to the login page mid-task. If they were filling out a long form, their work is lost.

**What to do instead:** Implement a two-token system. A short-lived access token (15 minutes to 1 hour) for API calls. A long-lived refresh token (7-30 days) stored in an httpOnly cookie. When the access token expires, the client automatically uses the refresh token to get a new access token. The user never notices.

```
Access Token: 15 minutes. Sent in Authorization header.
             If expired → use refresh token to get a new one.

Refresh Token: 30 days. Stored in httpOnly cookie.
              If expired → user must log in again.

When you get a 401:
  1. Try refreshing: POST /auth/refresh (sends refresh cookie automatically)
  2. If refresh succeeds: store new access token, retry the original request
  3. If refresh fails (refresh token expired): redirect to login
```

### 5. Overly permissive CORS settings

**What people do:** `Access-Control-Allow-Origin: *` because "it fixes the CORS error."

**Why it seems right:** "The error went away and everything works."

**What actually happens:** Any website can now make requests to your API while the user is logged in. If auth uses cookies (which the browser sends automatically), a malicious site can make requests to your API and the browser will include the user's auth cookies. This is essentially an open door. The attacker's site can read your user's data, make changes, anything your API allows.

**What to do instead:** Set `Access-Control-Allow-Origin` to your specific frontend domain(s). Never use `*` in production for APIs that use cookie-based auth.

```python
# BAD
app.add_middleware(CORSMiddleware, allow_origins=["*"])

# GOOD
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourapp.com", "http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)
```

### 6. Same error message for "user not found" and "wrong password"

**What people do:** Return "User not found" for invalid emails and "Wrong password" for invalid passwords.

**Why it seems right:** "It helps the user know what they got wrong."

**What actually happens:** An attacker can enumerate valid email addresses. They try random emails: "user not found," "user not found," "wrong password" (this email exists). Now they know which accounts exist and can target those with password attacks.

**What to do instead:** Return the same generic message for both cases: "Invalid email or password." The user experience is slightly worse (they don't know which field is wrong), but you've blocked account enumeration. The same applies to password reset: always say "If an account with that email exists, we've sent a reset link," whether or not the email is in your database.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Use an auth library for any production app.** NextAuth, Clerk, Supabase Auth, Lucia. Don't build auth from scratch. The security surface area is enormous and the edge cases are subtle.

2. **Store tokens in httpOnly cookies, not localStorage.** httpOnly cookies can't be read by JavaScript, which blocks XSS token theft. This is the single most important storage decision.

3. **Always verify JWT signatures with a specific algorithm.** Never accept `alg: none`. Never decode without verifying. Always specify `algorithms=["HS256"]` (or whichever algorithm you chose) explicitly.

4. **Use bcrypt (or argon2) for password hashing with at least 10 salt rounds.** Never store plain text passwords. Never use MD5 or SHA-256 for passwords (they're fast, which makes brute-force attacks fast). Use an algorithm designed to be slow.

5. **Implement token refresh for any app where sessions last more than an hour.** Short-lived access tokens (15-60 minutes) + long-lived refresh tokens (7-30 days) in httpOnly cookies. Refresh silently before the access token expires.

6. **Return the same error for "user not found" and "wrong password."** "Invalid email or password." Same for password reset: "If an account exists, we've sent a reset link." Prevents account enumeration.

7. **Set CORS to your specific domain(s), never `*`, for cookie-based auth.** Allow credentials only from origins you control. This prevents cross-site request attacks.

8. **Check authorization on every request, not just at login.** Authentication tells you who. Authorization tells you whether they're allowed. Check both. Especially for: admin routes, other users' data, and destructive actions.

## The "Just Tell Me What to Do" Quickstart

Set up Google OAuth login in a Next.js app with Auth.js v5 in under 10 minutes.

**Step 1: Create a Next.js app (skip if you have one)**

```bash
pnpm create next-app@latest auth-demo --typescript --tailwind --app --eslint
cd auth-demo
```

**Step 2: Install Auth.js**

```bash
pnpm add next-auth@beta
```

**Step 3: Get Google OAuth credentials**

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a project (or select one)
3. Go to APIs & Services > Credentials
4. Click "Create Credentials" > "OAuth Client ID"
5. Application type: Web application
6. Authorized redirect URIs: `http://localhost:3000/api/auth/callback/google`
7. Copy the Client ID and Client Secret

**Step 4: Set environment variables**

Create `.env.local`:

```
AUTH_SECRET=run-`npx auth secret`-to-generate-this
AUTH_GOOGLE_ID=your-google-client-id
AUTH_GOOGLE_SECRET=your-google-client-secret
```

Generate the AUTH_SECRET:

```bash
npx auth secret
```

**Step 5: Create auth configuration**

Create `auth.ts` in the project root:

```typescript
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [Google],
})
```

**Step 6: Create the API route**

Create `app/api/auth/[...nextauth]/route.ts`:

```typescript
import { handlers } from "@/auth"
export const { GET, POST } = handlers
```

**Step 7: Use it in a page**

Replace `app/page.tsx`:

```typescript
import { auth, signIn, signOut } from "@/auth"

export default async function Home() {
  const session = await auth()

  return (
    <main style={{ padding: "2rem", fontFamily: "system-ui" }}>
      {session ? (
        <div>
          <p>Signed in as {session.user?.name} ({session.user?.email})</p>
          {session.user?.image && (
            <img src={session.user.image} alt="" width={48} height={48} style={{ borderRadius: "50%" }} />
          )}
          <form action={async () => { "use server"; await signOut() }}>
            <button type="submit">Sign out</button>
          </form>
        </div>
      ) : (
        <div>
          <p>Not signed in</p>
          <form action={async () => { "use server"; await signIn("google") }}>
            <button type="submit">Sign in with Google</button>
          </form>
        </div>
      )}
    </main>
  )
}
```

**Step 8: Run it**

```bash
pnpm dev
```

Open `http://localhost:3000`. Click "Sign in with Google." After authorizing, you'll see your name, email, and avatar. That's OAuth, working, in under 10 minutes.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to implement auth, always include:
- The framework (Next.js, FastAPI, Express)
- The auth approach (session-based, JWT, OAuth)
- The auth library you're using (NextAuth, Clerk, Supabase Auth, Lucia, or none)
- What providers you need (Google, GitHub, email/password)
- Whether you need role-based access
- Whether this is for a web app, API, or both

### Example prompts that produce good results

**For OAuth setup:**
```
Set up Google OAuth in my Next.js 15 app using Auth.js v5.
I need: Google provider configuration, the API route handler,
a server component that shows login/logout based on session state,
and a middleware that protects all routes under /dashboard.
Use server actions for sign-in and sign-out.
```

**For protected API routes:**
```
Add JWT authentication to my FastAPI app. I need:
- POST /auth/login that takes email + password and returns access + refresh tokens
- A dependency that validates the JWT from the Authorization header
- Token refresh endpoint POST /auth/refresh
- Access token expires in 15 minutes, refresh token in 30 days
- Protect all routes under /api/v1 except /auth/login and /auth/refresh
Use python-jose for JWT. bcrypt for password hashing.
```

**For RBAC:**
```
Add role-based access control to my Next.js app with Auth.js.
Roles: admin, editor, viewer. Store the role in the session.
Create a middleware that checks roles. Admin can access /admin/*.
Editor can access /editor/* and /viewer/*. Viewer can only access
/viewer/*. Return 403 for unauthorized role access.
```

### What to watch out for in AI-generated auth code

- **localStorage for JWTs.** AI defaults to this. Always insist on httpOnly cookies or in-memory storage.
- **Missing signature verification.** AI sometimes decodes JWTs without verifying. Ensure the verify/decode call includes the secret key.
- **No CSRF protection.** AI often skips CSRF tokens for session-based auth. SameSite cookies help but aren't sufficient for all scenarios.
- **Hardcoded secrets.** AI puts secrets directly in the code instead of environment variables.
- **Missing error handling for expired tokens.** AI implements login but forgets token refresh.
- **Permissive CORS.** AI adds `allow_origins=["*"]` to make things work during development. This must be changed for production.

### Key terms that improve output quality

- "Use httpOnly cookies for token storage" (gets secure storage)
- "Include token refresh flow" (gets complete auth lifecycle)
- "Add rate limiting to the login endpoint" (gets brute-force protection)
- "Return same error for invalid email and wrong password" (gets secure error handling)
- "Use bcrypt with 12 salt rounds" (gets proper password hashing)

## Ship It: Build This

### Next.js App with Google OAuth, Protected Routes, and Role Check

Build a Next.js app where users log in with Google, some pages are protected (require login), and one page is admin-only.

**Why this project:** It exercises every concept in this guide. You'll implement OAuth (the full redirect flow via Auth.js), session management (how the user stays logged in), protected routes (middleware that checks auth), and authorization (admin vs regular user). You'll also deal with the practical reality of matching Google OAuth users to your database and assigning roles.

**Rough architecture:**
- Next.js 15 with App Router
- Auth.js v5 with Google provider
- SQLite database (via Prisma or Drizzle) for user records and roles
- Three page types: public (landing), protected (dashboard), admin-only (admin panel)
- Middleware that redirects unauthenticated users to login
- Role stored in database, added to the session via Auth.js callbacks
- First user to sign up gets admin role, all others get viewer role

**Key features to build:**
- Google OAuth sign-in and sign-out
- Session display (show user name, email, avatar, role)
- Protected route middleware (redirects to login if not authenticated)
- Admin-only page that returns 403 for non-admins
- User list page (admin only) showing all registered users and their roles
- Role upgrade/downgrade (admin can change other users' roles)
- Sign-out that clears the session completely

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn PKCE (Proof Key for Code Exchange)** when you need OAuth in a single-page app or mobile app where you can't keep a client secret. PKCE adds a cryptographic challenge that prevents authorization code interception.
- **Learn passkeys / WebAuthn** when you want passwordless login using biometrics (fingerprint, Face ID). This is where consumer auth is heading. Apple, Google, and Microsoft all support it.
- **Learn row-level security (RLS)** when you use Supabase and want the database itself to enforce "users can only see their own data." RLS moves authorization from your API code into the database, which is harder to bypass.
- **Learn multi-tenancy auth patterns** when your app serves multiple organizations and users belong to teams/workspaces with different roles in each.
- **Learn OAuth scopes and token management** when you need to access third-party APIs on behalf of users (reading their Google Calendar, posting to their Slack) beyond just login.
