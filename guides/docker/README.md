# Docker: Zero to Ship

> A box that holds your app and everything it needs to run, so it works the same everywhere.

## Why Should You Care?

You build something on your laptop. It works perfectly. You deploy it to a server and it breaks. The Node.js version is different. A system library is missing. An environment variable isn't set. The database driver expects a different OS. You spend three hours debugging something that has nothing to do with your actual code.

This is the most common problem in software development, and it's been around since the first person tried to move code from one computer to another. Docker solves it completely. You define exactly what your app needs, package it into a container, and that container runs identically on your laptop, your teammate's laptop, a CI server, and a production cloud. No "it works on my machine" conversations. No environment setup docs that are always outdated. No surprise runtime differences.

If you're building with AI tools, Docker becomes even more important. Cursor and Claude Code generate code that assumes certain dependencies exist. They'll scaffold a project with sharp (an image library that needs native binaries), or use a Postgres client that needs specific shared libraries. Locally, you've installed these over time and forgotten about them. On a fresh server, nothing works. A Dockerfile makes those invisible dependencies explicit. It's the recipe that turns "it runs on my machine" into "it runs."

## The 30-Second Version

Docker packages your application and all its dependencies (runtime, libraries, system tools, config files) into a single unit called a container. You write a Dockerfile that describes how to build your app. Docker uses that file to create an image (a snapshot). You run the image and get a container (a running instance). The container is isolated from your host machine, uses its own filesystem, and behaves identically everywhere. Docker Compose lets you run multiple containers together (your app, a database, a cache) with one command.

## The Real Explanation (ELI5)

Imagine you're a chef who makes a signature dish. At your restaurant, you have the perfect kitchen: the right pans, the right spices, the right oven temperature. You know where everything is.

Now someone invites you to cook at their house. Their kitchen is different. They don't have the right pan. Their oven runs hot. They're missing two spices. You can still cook, but you spend half the time adapting to the kitchen instead of actually cooking.

What if you could bring your entire kitchen with you? Not just the ingredients, but the pans, the oven, the spices, the counter space, everything. You show up, unfold your kitchen, cook exactly the way you do at home, and the dish comes out perfect.

That's Docker. Your app is the recipe. The Dockerfile is the packing list for what goes in your portable kitchen. The image is the packed-up kitchen, ready to ship. The container is the kitchen, unfolded and running somewhere.

Now replace "kitchen" with "environment" (operating system, runtime, libraries), "recipe" with "application code," and "cooking at someone else's house" with "deploying to a server." That's the whole idea.

The analogy breaks at one important point: real kitchens are expensive to duplicate. Docker containers are cheap. You can run ten containers on a single laptop. Starting one takes seconds, not hours. Throwing one away and starting fresh is trivial. This changes how you think about environments: they become disposable.

## How It Actually Works

### The Three Core Concepts

```
Dockerfile ──build──> Image ──run──> Container

(recipe)              (frozen meal)   (meal being served)
```

**Dockerfile:** A text file with instructions. "Start with Ubuntu. Install Node.js. Copy my code. Install dependencies. Run this command." Each instruction creates a layer.

**Image:** The result of building a Dockerfile. It's a read-only snapshot of the filesystem. Immutable. You can share images through registries (Docker Hub is the biggest one, like GitHub but for images).

**Container:** A running instance of an image. It has its own filesystem, its own network, its own process space. You can run multiple containers from the same image, just like you can bake multiple cookies from the same cookie cutter.

### What happens when you run `docker build`

```
Step 1/6: FROM node:20-alpine
  → Pull the Node.js 20 base image (Alpine Linux, minimal)

Step 2/6: WORKDIR /app
  → Create and switch to /app directory inside the image

Step 3/6: COPY package*.json ./
  → Copy package.json and package-lock.json into the image

Step 4/6: RUN npm install
  → Install dependencies (this layer gets cached)

Step 5/6: COPY . .
  → Copy the rest of your application code

Step 6/6: CMD ["node", "server.js"]
  → Set the default command to run when the container starts
```

**Under the hood:** Each step creates a layer. Layers are cached. If you change your application code but not package.json, Docker skips steps 1-4 and only re-runs 5-6. This makes rebuilds fast. The order of instructions in your Dockerfile directly affects build speed.

**Why it's designed this way:** Layered caching means you're not reinstalling every dependency every time you change a line of code. The tradeoff is that layer order matters, and getting it wrong means slow builds. But the pattern is simple once you learn it: things that change less go first.

### What happens when you run `docker run`

```
Your Machine                           Container
┌─────────────────┐                   ┌──────────────────┐
│                  │                   │  Alpine Linux     │
│  macOS / Windows │   docker run     │  Node.js 20       │
│  / Linux         │ ──────────────>  │  Your app code    │
│                  │                   │  node_modules/    │
│  Port 3000 ─────────── mapped ──────── Port 3000       │
│                  │                   │                    │
│  ./data ─────────────── volume ─────── /app/data        │
│                  │                   │                    │
└─────────────────┘                   └──────────────────┘
```

The container runs in isolation. It has its own filesystem, its own network stack, its own process tree. Your Mac doesn't know or care that there's a Linux system running inside it.

**Port mapping** connects a port on your machine to a port inside the container. `-p 3000:3000` means "when I visit localhost:3000, send that traffic to port 3000 inside the container." The two numbers don't have to match: `-p 8080:3000` maps your port 8080 to the container's port 3000.

**Volumes** connect a folder on your machine to a folder inside the container. This is how data persists when a container is destroyed and recreated. Without a volume, everything inside the container disappears when it stops.

## The Mental Model

**Think of an image as a frozen meal and a container as the meal being heated up.** The frozen meal (image) is the same every time. You can make as many servings (containers) as you want from it. The meal being heated up (container) is temporary. When you're done, you throw it away. You don't try to re-freeze it. This means whenever you see a bug, the answer is almost always "fix the Dockerfile, rebuild the image, start a fresh container." Not "SSH into the container and fix it."

**Think of a Dockerfile as a recipe, where each line is one step, and the order matters because each step builds on the last.** This means whenever you're optimizing a Dockerfile, you're really asking "which steps change most often?" Those go last, so cache hits on the earlier steps save you time.

**Think of Docker Compose as a blueprint for a building, not just one room.** Your app doesn't run alone. It needs a database, maybe a cache, maybe a message queue. Compose lets you describe the entire building (all services, how they connect, what volumes they share) in one file. `docker compose up` builds the whole thing. This means whenever you're setting up a local development environment, Docker Compose should be your first thought.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Image | A snapshot of a filesystem with everything your app needs. Read-only. | Every time you build or pull. |
| Container | A running instance of an image. Isolated, temporary. | Every time you start an app. |
| Dockerfile | A text file with step-by-step instructions to build an image. | In the root of every Docker project. |
| Layer | One instruction in a Dockerfile creates one layer. Layers are cached. | When you're debugging slow builds. |
| Registry | A place to store and share images. Docker Hub is the default. | When pulling base images or pushing your own. |
| Base image | The starting point for your Dockerfile. Usually an OS with a runtime pre-installed. | The FROM line in every Dockerfile. |
| Alpine | A tiny Linux distribution (5MB). Used as a base to keep images small. | `node:20-alpine`, `python:3.12-alpine`. |
| Volume | A folder shared between your machine and a container, or persisted between container restarts. | When your app needs to save files or you want hot reload. |
| Port mapping | Connecting a port on your machine to a port inside the container. | The `-p` flag in `docker run`. |
| Docker Compose | A tool for defining and running multiple containers together using a YAML file. | Any project with more than one service (app + database). |
| Build context | The folder Docker sends to the build process. Everything in it is available to COPY. | When your builds are slow because you're sending too many files. |
| `.dockerignore` | A file listing what to exclude from the build context. Like `.gitignore` for Docker. | In the root of your project, next to the Dockerfile. |
| Multi-stage build | A Dockerfile with multiple FROM statements. Build in one stage, copy the result to a smaller stage. | When you need a small production image. |
| Tag | A label on an image, usually a version number. `node:20` and `node:20-alpine` are different tags. | When pulling images or specifying base images. |
| `CMD` | The default command a container runs when it starts. | The last line of most Dockerfiles. |
| `ENTRYPOINT` | Like CMD, but harder to override. Used when the container should always run a specific executable. | In base images and production Dockerfiles. |
| `EXPOSE` | Documents which port the container listens on. Doesn't actually open the port. | In Dockerfiles, as documentation for other developers. |

## Common Patterns

### Pattern 1: Node.js App with Dockerfile

**When to use it:** You have a Node.js API or app and want to deploy it consistently anywhere.

**How it works:** Start from a Node.js base image, install dependencies with caching, copy code, and set the start command.

```dockerfile
# Use a specific version. Never use "latest".
FROM node:20-alpine

WORKDIR /app

# Copy dependency files first (caching layer)
COPY package.json package-lock.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy application code (changes most often, so goes last)
COPY . .

# Document the port (doesn't open it)
EXPOSE 3000

# Start the app
CMD ["node", "server.js"]
```

Build and run:

```bash
docker build -t my-app .
docker run -p 3000:3000 -e DATABASE_URL=postgres://... my-app
```

### Pattern 2: Full-Stack App with Docker Compose

**When to use it:** Your app needs multiple services (web server, database, cache) and you want to start everything with one command.

**How it works:** A `docker-compose.yml` file defines all services, their images, ports, volumes, and environment variables. Docker Compose creates a network so services can talk to each other by name.

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/myapp
      REDIS_URL: redis://cache:6379
    depends_on:
      - db
      - cache

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

The key detail: `db` and `cache` in the connection URLs aren't hostnames you configure. They're the service names from this file. Docker Compose creates a network where services find each other by name.

```bash
docker compose up        # start everything
docker compose up -d     # start in background
docker compose down      # stop everything
docker compose down -v   # stop everything and delete volumes (destroys data)
```

### Pattern 3: Development Environment with Hot Reload

**When to use it:** You want Docker for consistency but still need instant feedback when you change code.

**How it works:** Mount your source code as a volume so changes on your machine immediately appear inside the container. Use a dev command (like `nodemon` or `next dev`) instead of the production start command.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app           # Mount source code
      - /app/node_modules # But DON'T mount node_modules (use container's copy)
    environment:
      NODE_ENV: development
```

The `Dockerfile.dev`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install       # all deps, including devDependencies
COPY . .
CMD ["npx", "nodemon", "server.js"]
```

The trick: `.:/app` mounts your entire project into the container, but `/app/node_modules` creates an anonymous volume that keeps the container's own node_modules. This prevents your Mac's node_modules (compiled for macOS) from overwriting the container's node_modules (compiled for Linux).

### Pattern 4: CI/CD Pipeline with Docker

**When to use it:** You want your CI pipeline (GitHub Actions, GitLab CI) to build and push Docker images on every merge.

**How it works:** CI builds the image, tags it with the git SHA or version, and pushes it to a registry. Deployment pulls the image and runs it.

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            myuser/myapp:latest
            myuser/myapp:${{ github.sha }}
```

Tag with the git SHA so you can always trace a running container back to the exact commit that produced it.

### Pattern 5: Multi-Stage Build for Production

**When to use it:** Your production image should be as small as possible. No dev dependencies, no build tools, no source maps.

**How it works:** Use multiple FROM statements. The first stage installs everything and builds. The second stage starts fresh and copies only what's needed to run.

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/package.json /app/package-lock.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

Result: The final image has no TypeScript compiler, no testing libraries, no source code. Just the compiled output and production dependencies. Image size drops from 500MB+ to under 150MB.

## Mistakes Everyone Makes

### 1. Running as root in containers

**What people do wrong:** They don't specify a user in the Dockerfile, so the app runs as root inside the container.

**Why it seems right:** It works. No permission errors. Everything has access to everything.

**What actually happens:** If an attacker exploits a vulnerability in your app, they have root access inside the container. Container isolation is good but not perfect. Root in a container can sometimes escape to root on the host, especially with misconfigurations.

**What to do instead:**

```dockerfile
# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to that user
USER appuser

# Everything after this runs as appuser
COPY --chown=appuser:appgroup . .
CMD ["node", "server.js"]
```

### 2. Not using .dockerignore (huge images and leaked secrets)

**What people do wrong:** They don't create a `.dockerignore` file, so `COPY . .` sends everything to Docker: node_modules, .git, .env files, test fixtures, IDE config.

**Why it seems right:** `COPY . .` should copy the code, right?

**What actually happens:** Your build context balloons. A project with a 200MB node_modules folder sends all 200MB to Docker before the build even starts. Worse, your `.env` file with secrets ends up baked into the image. Anyone who pulls the image can extract those secrets.

**What to do instead:** Create a `.dockerignore` in your project root:

```
node_modules
.git
.env
.env.*
*.log
dist
.next
coverage
.DS_Store
```

### 3. Putting COPY before dependency install (cache busting)

**What people do wrong:**

```dockerfile
COPY . .
RUN npm install
```

**Why it seems right:** Copy everything, then install. Logical order.

**What actually happens:** Every time you change any file in your project (even a comment in a README), Docker invalidates the COPY layer. Since `npm install` comes after, it also gets invalidated. You reinstall every dependency on every single code change. A build that should take 5 seconds takes 2 minutes.

**What to do instead:**

```dockerfile
COPY package.json package-lock.json ./
RUN npm install
COPY . .
```

Dependency files rarely change. By copying them first, the `npm install` layer stays cached until you actually change a dependency.

### 4. Not using multi-stage builds for production

**What people do wrong:** They use one Dockerfile that installs everything (TypeScript compiler, testing libraries, build tools) and ship the whole thing to production.

**Why it seems right:** It's simpler. One file. One build. It works.

**What actually happens:** Your production image is 1GB+ instead of 150MB. It takes longer to push, pull, and deploy. It has a massive attack surface because it includes tools (compilers, package managers, dev tools) that should never exist in production.

**What to do instead:** Use a multi-stage build (see Pattern 5 above). Build stage has everything. Production stage has only what's needed to run.

### 5. Using `latest` tag instead of pinned versions

**What people do wrong:**

```dockerfile
FROM node:latest
```

**Why it seems right:** You always get the newest version. Sounds good.

**What actually happens:** Your build works today with Node 20. Next month, `latest` points to Node 22. Your build breaks because a dependency doesn't support Node 22 yet. Or worse, it doesn't break, but behavior changes subtly and you don't notice until production. You can't reproduce the exact build from last week because `latest` has moved.

**What to do instead:**

```dockerfile
FROM node:20-alpine
```

Pin the major version. Use `-alpine` for smaller images. Update the version intentionally when you're ready to test against it.

### 6. Storing secrets in the image

**What people do wrong:**

```dockerfile
ENV DATABASE_URL=postgres://user:password@host:5432/db
```

Or worse, `COPY .env .`.

**Why it seems right:** The app needs these values to run. Put them in the image.

**What actually happens:** Anyone who has access to the image can run `docker inspect` or `docker history` and see every environment variable and every file you copied in. Images are often stored in shared registries. Your database password is now visible to everyone with pull access.

**What to do instead:** Pass secrets at runtime, never at build time:

```bash
docker run -e DATABASE_URL=postgres://... my-app
```

Or use Docker Compose with an `.env` file that's in `.gitignore` and `.dockerignore`:

```yaml
services:
  app:
    env_file: .env
```

### 7. Not understanding the build context

**What people do wrong:** They put their Dockerfile deep in a subfolder and wonder why `COPY ../shared ./shared` doesn't work.

**Why it seems right:** Relative paths work everywhere else.

**What actually happens:** Docker can only see files within the build context (the directory you pass to `docker build`). If you run `docker build .` from `/app/backend`, the Dockerfile can only COPY files from `/app/backend` and below. It can't reach `../shared`.

**What to do instead:** Run the build from a parent directory and point to the Dockerfile:

```bash
docker build -f backend/Dockerfile .
```

Now the build context is the current directory (the monorepo root), and the Dockerfile can COPY from anywhere within it.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Pin your base image versions.** `node:20-alpine`, not `node:latest`. Reproducible builds prevent mysterious production failures.

2. **Order Dockerfile instructions from least-changing to most-changing.** System deps first, then language deps, then your code. This maximizes layer cache hits and keeps builds fast.

3. **One process per container.** Don't run your app and your database in the same container. Use Compose for multi-service setups. This makes each piece independently scalable, restartable, and debuggable.

4. **Use `.dockerignore` in every project.** At minimum, exclude `node_modules`, `.git`, `.env`, and build artifacts. It keeps images small and prevents secret leakage.

5. **Never store secrets in images.** Pass them as environment variables at runtime or use Docker secrets/Compose env files. Secrets baked into images are visible to anyone who pulls them.

6. **Use multi-stage builds for production.** Build in one stage, copy the output to a minimal stage. The final image should have nothing that isn't needed to run.

7. **Run as a non-root user.** Add a `USER` instruction after installing dependencies. It limits the damage if your container is compromised.

8. **Use `npm ci` instead of `npm install` in Dockerfiles.** `ci` installs from the lockfile exactly, is faster, and won't surprise you with different versions than what you tested locally.

## The "Just Tell Me What to Do" Quickstart

You'll containerize a basic Node.js app and run it with a Postgres database using Docker Compose. Total time: under 10 minutes.

**Prerequisites:** Docker Desktop installed. ([Download here](https://docs.docker.com/get-docker/) if not.)

**Step 1: Create a project folder**

```bash
mkdir docker-quickstart && cd docker-quickstart
```

**Step 2: Create a simple Node.js app**

```bash
npm init -y
npm install express pg
```

Create `server.js`:

```js
const express = require("express");
const { Pool } = require("pg");

const app = express();
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

app.get("/", async (req, res) => {
  try {
    const result = await pool.query("SELECT NOW() as time");
    res.json({ message: "Hello from Docker!", dbTime: result.rows[0].time });
  } catch (err) {
    res.status(500).json({ error: "Database not ready yet" });
  }
});

app.listen(3000, () => console.log("Server running on port 3000"));
```

**Step 3: Create a Dockerfile**

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --only=production

COPY . .

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

**Step 4: Create a .dockerignore**

```
node_modules
.git
.env
*.log
```

**Step 5: Create docker-compose.yml**

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/quickstart
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: quickstart
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Step 6: Run it**

```bash
docker compose up --build
```

Wait for the logs to settle. Visit `http://localhost:3000`. You should see a JSON response with the current time from Postgres.

**Step 7: Verify the database persists**

```bash
docker compose down         # stop everything
docker compose up -d        # restart in background
curl http://localhost:3000   # still works, data persisted via volume
```

**Step 8: Clean up**

```bash
docker compose down -v      # stop and delete volumes
```

That's it. You have a containerized Node.js app connected to Postgres, with data persistence, proper layer caching, non-root user, and a production-ready Dockerfile structure.

## How to Prompt AI Tools About This

### Context to provide

When asking Claude, Cursor, or Copilot to help with Docker, always specify:
- Your runtime and version ("Node.js 20", "Python 3.12")
- What services you need ("Postgres, Redis")
- Whether it's for development or production
- Your OS (Alpine images behave differently from Debian-based ones)

### Example prompts that produce good results

**Basic Dockerfile:**
> "Write a Dockerfile for a Node.js 20 Express API. Use alpine base, multi-stage build, non-root user, proper layer caching for npm dependencies. Production only, no dev dependencies in final image."

**Docker Compose:**
> "Write a docker-compose.yml for local development: Next.js app (port 3000), Postgres 16 (port 5432 with persistent volume), Redis 7 (port 6379). Include health checks for the database. App should wait for db to be healthy before starting."

**Debugging:**
> "My Docker build is slow. Here's my Dockerfile: [paste]. Why is npm install running every time I change a source file? How do I fix the layer caching?"

**Migration from local to Docker:**
> "I have a working Node.js app that uses Postgres and Redis locally. Here's my package.json: [paste]. Create a Dockerfile and docker-compose.yml that replaces my local setup. Include volume mounts for hot reload in development."

### What to watch out for in AI-generated Docker code

- **Missing `.dockerignore`.** AI almost never generates this. Always ask for it explicitly or create one yourself.
- **Using `npm install` instead of `npm ci`.** AI defaults to `install`. Ask for `ci` in production Dockerfiles.
- **No non-root user.** Most AI-generated Dockerfiles run as root. Always check.
- **`FROM node:latest`.** AI loves `latest`. Always change it to a pinned version.
- **Secrets in ENV instructions.** AI will sometimes hardcode database passwords in the Dockerfile. Move these to runtime environment variables.
- **No health checks.** For Docker Compose, ask specifically for health checks on databases so your app doesn't start before the database is ready.

### Key terms to use in prompts

Using these terms gets noticeably better output: "multi-stage build," "layer caching," "non-root user," "alpine base," "health check," "named volume," "build context," "production-ready."

## Ship It: Build This

Dockerize an existing Node.js application with a Postgres database and Redis cache, using Docker Compose for orchestration.

**What you're building:** A task management API with three services running in containers. The API handles CRUD operations for tasks stored in Postgres. Redis caches frequently accessed task lists. Docker Compose ties everything together. You'll have separate Dockerfiles for development (with hot reload) and production (multi-stage, minimal image).

**Why this project specifically:** It exercises every important Docker concept: multi-stage builds, volume mounting, inter-service networking, data persistence, environment variable management, layer caching, and the dev-vs-production Dockerfile split. It's also a project you can extend into a real app.

**Rough architecture:**

- `Dockerfile` for production (multi-stage, alpine, non-root user, ~150MB final image)
- `Dockerfile.dev` for development (full deps, nodemon, volume-mounted source)
- `docker-compose.yml` for local development (app + postgres + redis, hot reload)
- `docker-compose.prod.yml` for production-like local testing
- `.dockerignore` to keep images clean
- Health checks on Postgres and Redis so the app waits for dependencies
- Named volumes for Postgres data persistence
- Environment variables for all configuration (no hardcoded values)

**Estimated time:** 2-3 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn Docker networking** when you need containers across different Compose files to talk to each other, or when you're debugging "connection refused" errors between services.
- **Learn Docker health checks in depth** when you need to verify that a service is not just running but actually ready to accept connections (critical for databases and message queues).
- **Learn Docker build caching strategies** when your CI builds are slow and you want to use BuildKit cache mounts or registry-based caching to speed them up.
- **Learn container orchestration (Kubernetes, Docker Swarm)** when you're running multiple instances of your app across multiple servers and need automated scaling, rolling deploys, and self-healing.
- **Learn Docker security scanning** when you're deploying to production and need to catch known vulnerabilities in your base images and dependencies before they're exploitable.
