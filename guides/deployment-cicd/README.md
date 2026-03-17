# Deployment and CI/CD: Zero to Ship

> Getting your code from your laptop to a place where anyone in the world can use it, automatically, every time you push.

## Why Should You Care?

You've built something. It works on your laptop. Now what? Someone needs to be able to type a URL into their browser and use it. That gap between "works locally" and "works for users" is deployment. And closing that gap manually, every single time you make a change, is a recipe for broken production, missed steps, and 2 AM panics.

CI/CD automates that gap. Continuous Integration means every code change gets tested automatically. Continuous Deployment means every change that passes tests gets shipped automatically. Together, they turn "I need to spend 30 minutes deploying" into "I push to main and it's live in 90 seconds." For solo builders and small teams, this isn't a luxury. It's the difference between shipping once a week (because deploying is painful) and shipping ten times a day (because deploying is invisible).

If you're building with AI tools, you're generating code fast. Cursor and Claude Code can scaffold an entire feature in minutes. But speed of writing code is useless without speed of shipping code. A solid CI/CD pipeline means you can trust that AI-generated code gets tested before it touches production, that broken deployments get rolled back, and that you spend your time building features instead of babysitting deploys.

## The 30-Second Version

Deployment is copying your code to a server and starting it. CI/CD automates this: every time you push code, a pipeline runs your tests (CI), and if they pass, deploys automatically (CD). Platforms like Vercel and Netlify handle everything for frontend apps, just connect your GitHub repo. For backend services, tools like Railway and Fly.io run your Docker containers. GitHub Actions is the most common way to define CI/CD pipelines: a YAML file in your repo that tells GitHub what to run, when to run it, and what to do with the results. Good deployment pipelines are invisible when everything works and loud when something breaks.

## The Real Explanation (ELI5)

Imagine you write a newspaper column. You write it on your typewriter at home, then you need to get it into tomorrow's paper.

In the old days, you'd drive to the printing press, hand over your manuscript, wait while they typeset it, proofread the proof, approve it, and watch it roll off the press. If you made a typo, you'd have to drive back and start over. You'd do this for every single column.

Now imagine a better system. You write your column and drop it in a special mailbox outside your house. A courier picks it up and takes it to the newspaper office. At the office, an editor reads it and checks for errors (that's CI, continuous integration). If the editor finds problems, they call you and you fix it before it goes further. If it passes, it goes straight to the printing press and into tomorrow's paper (that's CD, continuous deployment). You never drive to the office. You never wait at the press. You write, you drop it in the mailbox, and the system handles the rest.

Now replace "mailbox" with "git push," "courier" with "GitHub Actions," "editor" with "automated tests," and "printing press" with "Vercel/Railway/your hosting platform." That's CI/CD.

The analogy extends: what if you want to preview the column before it goes to print? That's a preview deployment, a temporary version that only you and your editor can see. What if yesterday's column had a factual error and you need to pull it? That's a rollback, reverting to the previous version. What if you want to know the moment a reader complains? That's monitoring.

The analogy breaks at scale. A newspaper prints once a day. With CI/CD, you can "print" fifty times a day. Each version is tracked, reversible, and independent. That changes how you think about shipping: small, frequent changes instead of big, scary releases.

## How It Actually Works

### The Deployment Spectrum

Not every app needs the same deployment setup. Think of it as a spectrum from fully managed to fully manual:

```
Fully Managed ◄──────────────────────────────────────────► Fully Manual

Vercel         Railway        AWS ECS        AWS EC2
Netlify        Fly.io         Google Cloud   Bare metal
Cloudflare     Render         Azure          VPS (DigitalOcean)

"Push and forget"   "Config and forget"   "Build your own"
```

**Static/Jamstack hosting (Vercel, Netlify, Cloudflare Pages):**
- Best for: Next.js, static sites, frontend apps
- You push code. They build it, deploy it to a global CDN, give you a URL.
- Zero server management. Free tier is generous.
- Limitation: backend logic runs as serverless functions with cold starts, timeout limits, and no persistent connections.

**Container platforms (Railway, Fly.io, Render):**
- Best for: Backend APIs, full-stack apps, anything that needs a persistent process
- You give them a Dockerfile (or they detect your framework). They build and run it.
- More control than Vercel. You can run background jobs, WebSocket servers, cron tasks.
- You manage environment variables, scaling, and database connections.

**Cloud providers (AWS, GCP, Azure):**
- Best for: Complex architectures, specific compliance requirements, very high scale
- Maximum control. You configure networking, load balancers, auto-scaling, IAM permissions.
- Steep learning curve. Easy to overspend. Most startups and solo builders don't need this.

**The honest answer:** Start with Vercel for frontend and Railway or Fly.io for backend. Move to AWS/GCP when you have a specific reason (compliance, scale, cost optimization at high volume). Most apps never need to make that move.

### How Vercel/Netlify Actually Work

When you connect your GitHub repo to Vercel and push to main:

```
git push origin main
        │
        ▼
GitHub sends webhook to Vercel
        │
        ▼
Vercel clones your repo
        │
        ▼
Vercel runs your build command (next build)
        │
        ▼
Build output is optimized:
  ├── Static pages → uploaded to CDN (300+ edge locations)
  ├── Server components → deployed as serverless functions
  ├── API routes → deployed as serverless functions
  └── Assets (images, CSS, JS) → uploaded to CDN with cache headers
        │
        ▼
DNS updated to point to new deployment
        │
        ▼
Previous deployment kept (instant rollback available)
        │
        ▼
Live in 30-90 seconds
```

**What happens when a user visits your site:**

```
User in Mumbai                    User in New York
      │                                  │
      ▼                                  ▼
Edge node in Mumbai              Edge node in Virginia
(serves cached static pages)     (serves cached static pages)
      │                                  │
      └──── only if needed ──────────────┘
                    │
                    ▼
          Serverless function
          (runs your API routes,
           server components)
          in nearest region
```

**Under the hood:** Vercel doesn't keep a server running for your app. Static pages and assets live on a CDN (Content Delivery Network), a network of servers around the world that serve files from the closest location to the user. Dynamic parts (API routes, server components) run as serverless functions that spin up on demand, handle the request, and shut down. You don't pay for idle time. You pay for execution time.

**Why it's designed this way:** Most web traffic is serving the same HTML, CSS, and JavaScript to many users. Putting that on a CDN means it loads fast everywhere. The dynamic parts (database queries, authentication) only run when needed. This architecture is cheap at low traffic and scales automatically at high traffic. The tradeoff is that serverless functions have cold starts (first request after idle is slower) and timeout limits (usually 10-30 seconds).

### Environment Variables in Production

Locally, you have a `.env` file:

```
DATABASE_URL=postgres://localhost:5432/myapp
API_KEY=sk-test-12345
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

In production, these values are different. The database is on a remote server, the API key is a production key, and the app URL is your actual domain. You don't commit production values to git, ever.

**Where environment variables live:**

| Platform | Where to set them | How they work |
|----------|-------------------|---------------|
| Vercel | Project Settings > Environment Variables | Set per environment (Production, Preview, Development). Available at build time and runtime. |
| Netlify | Site Settings > Environment Variables | Same as Vercel. Available at build time. |
| Railway | Service > Variables | Injected at runtime. Can reference other services (e.g., `${{Postgres.DATABASE_URL}}`). |
| Fly.io | `fly secrets set KEY=value` | Encrypted at rest. Injected at runtime. |
| GitHub Actions | Repository Settings > Secrets | Available in workflow files as `${{ secrets.KEY }}`. Never printed in logs. |

**The critical distinction:**

```
NEXT_PUBLIC_API_URL=...    ← Prefix means this is bundled into client JavaScript.
                              Anyone can see it. Only use for public values.

DATABASE_URL=...           ← No prefix. Server-only. Never sent to the browser.
                              Safe for secrets.
```

In Next.js, any variable starting with `NEXT_PUBLIC_` gets inlined into the JavaScript bundle at build time. This means the value is baked into the code that runs in the user's browser. Never put API keys, database URLs, or any secret in a `NEXT_PUBLIC_` variable.

### CI/CD: What It Actually Means

**Continuous Integration (CI):** Every time someone pushes code or opens a PR, automated checks run. Linting, type checking, tests, security scans. The code isn't merged until all checks pass. This catches bugs before they reach main.

**Continuous Deployment (CD):** Every time code lands on main (after passing CI), it's automatically deployed to production. No manual steps. No "deploy button" someone has to remember to click.

**Continuous Delivery (also CD, confusingly):** Same as continuous deployment, except there's a manual approval step before production. Code is always ready to deploy, but a human clicks the button. Common in teams with compliance requirements.

```
Developer pushes code
        │
        ▼
    ┌─────────────────────────┐
    │    Continuous Integration │
    │                           │
    │  ✓ Code compiles          │
    │  ✓ Linting passes         │
    │  ✓ Type checking passes   │
    │  ✓ Tests pass             │
    │  ✓ Security scan clean    │
    └─────────┬───────────────┘
              │
        Pass? ──── No ──→ PR blocked, developer notified
              │
             Yes
              │
              ▼
    ┌─────────────────────────┐
    │  Continuous Deployment    │
    │                           │
    │  Build production bundle  │
    │  Deploy to hosting        │
    │  Run smoke tests          │
    │  Notify team              │
    └─────────────────────────┘
              │
              ▼
         Live in production
```

### GitHub Actions: Anatomy of a Workflow

GitHub Actions is the most common CI/CD tool for projects hosted on GitHub. It's free for public repos and has a generous free tier for private repos (2,000 minutes/month).

A workflow is a YAML file in `.github/workflows/`. Here's every part explained:

```yaml
# .github/workflows/ci.yml

# Name shown in the GitHub Actions tab
name: CI

# When to run this workflow
on:
  push:
    branches: [main]          # Run on pushes to main
  pull_request:
    branches: [main]          # Run on PRs targeting main

# What to run
jobs:
  # Job name (you pick this)
  test:
    # What machine to run on
    runs-on: ubuntu-latest

    # Steps run sequentially within a job
    steps:
      # Check out your code
      - uses: actions/checkout@v4

      # Set up Node.js
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"       # Cache pnpm dependencies between runs

      # Install dependencies
      - run: pnpm install --frozen-lockfile

      # Run your checks
      - run: pnpm lint
      - run: pnpm type-check
      - run: pnpm test

  # Another job (runs in parallel with 'test' by default)
  build:
    runs-on: ubuntu-latest
    needs: test               # Wait for 'test' to pass first

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
```

**Key concepts:**

- **Triggers (`on`):** What causes the workflow to run. Push to a branch, PR opened, manual dispatch, cron schedule, or another workflow completing.
- **Jobs:** Independent units of work. Jobs run in parallel by default. Use `needs:` to create dependencies.
- **Steps:** Sequential commands within a job. Each step is either a `uses:` (a reusable action from the marketplace) or a `run:` (a shell command).
- **Runners (`runs-on`):** The machine your code runs on. `ubuntu-latest` is the most common. Also available: `macos-latest`, `windows-latest`.
- **Secrets:** Sensitive values stored in GitHub repository settings. Referenced as `${{ secrets.MY_SECRET }}`. Never printed in logs, even if you try.
- **Artifacts:** Files produced by one job that another job needs. Uploaded with `actions/upload-artifact` and downloaded with `actions/download-artifact`.

### Preview Deployments

Every PR should get its own URL. Here's why.

Without preview deployments, reviewing a PR means: check out the branch, install dependencies, run the dev server, manually test. Most reviewers skip this and just read the code diff. Bugs that are obvious when you see the UI go unnoticed.

With preview deployments, every PR automatically gets a live URL. The reviewer clicks the link and sees the actual change running. QA can test it. The designer can check the UI. The product manager can try the feature. All without touching their terminal.

**How it works on Vercel:**

```
Developer opens PR
        │
        ▼
Vercel detects PR via GitHub webhook
        │
        ▼
Builds the PR branch
        │
        ▼
Deploys to a unique URL:
  https://my-app-abc123-developers-projects.vercel.app
        │
        ▼
Posts the URL as a comment on the PR
        │
        ▼
Every new push to the PR branch triggers a new preview build
        │
        ▼
When PR is merged or closed, preview is deactivated
```

This is automatic on Vercel and Netlify. No configuration needed. For other platforms, you can set it up with GitHub Actions (deploy to a temporary environment on PR open, tear it down on PR close).

**Preview deployments use preview environment variables.** On Vercel, you can set different values for Production, Preview, and Development environments. This means your preview deployment can point to a staging database instead of production, use test API keys, and have debug logging enabled.

### Rollbacks

Production is broken. Users are seeing errors. What do you do?

**Option 1: Revert and redeploy (recommended)**

```bash
# Find the commit that broke things
git log --oneline -10

# Revert it (creates a new commit that undoes the changes)
git revert abc1234

# Push. CI/CD deploys the fix automatically.
git push origin main
```

This is the safest approach. The git history shows exactly what happened: a bad commit went out, it was reverted, and a fix was deployed. Everyone can see the timeline.

**Option 2: Instant rollback via platform**

Most platforms keep previous deployments available:

| Platform | How to rollback |
|----------|----------------|
| Vercel | Dashboard > Deployments > click any previous deployment > "Promote to Production" |
| Netlify | Dashboard > Deploys > click previous deploy > "Publish deploy" |
| Railway | Dashboard > Deployments > Rollback button |
| Fly.io | `fly releases` then `fly deploy --image <previous-image>` |

This is faster (seconds vs. minutes for a full CI/CD cycle) but doesn't fix the git history. Use it for emergencies, then follow up with a proper revert.

**Option 3: Feature flags**

For changes you're not sure about, wrap them in a feature flag:

```typescript
if (featureFlags.newCheckoutFlow) {
  return <NewCheckout />;
}
return <OldCheckout />;
```

If the new checkout breaks, flip the flag off. No deployment needed. The code stays in the codebase but doesn't execute. Tools like LaunchDarkly, Flagsmith, or even a simple database table make this easy.

**The rollback rule:** Every deployment to production should be reversible within 5 minutes. If it's not, you need a better rollback strategy before you ship.

### Monitoring Basics

Deploying is only half the job. You also need to know when something breaks, ideally before users tell you.

**Three layers of monitoring:**

```
Layer 1: Is it up?
  └── Uptime monitoring (Pingdom, UptimeRobot, Better Stack)
      Pings your URL every 30-60 seconds.
      Alerts you (email, Slack, SMS) if it's down.

Layer 2: Are there errors?
  └── Error tracking (Sentry, LogRocket, Highlight.io)
      Captures unhandled exceptions with full stack traces.
      Groups duplicate errors. Shows which deploy introduced them.
      Includes user context (browser, URL, user ID).

Layer 3: Is it slow?
  └── Performance monitoring (Vercel Analytics, Sentry Performance)
      Tracks page load times, API response times, Core Web Vitals.
      Alerts when response times exceed thresholds.
```

**The minimum viable monitoring setup:**

1. **Uptime check:** Use UptimeRobot (free tier: 50 monitors). Create a monitor for your production URL. Set up Slack or email alerts.

2. **Error tracking:** Add Sentry to your app. It takes 5 minutes to set up and catches every unhandled error with context. The free tier handles 5,000 events/month.

3. **Deploy notifications:** Send a message to Slack/Discord when a deploy succeeds or fails. This creates a timeline you can correlate with error spikes.

```
14:32 - Deploy #47 succeeded (commit abc123: "add new payment flow")
14:35 - Error spike: "Cannot read property 'id' of undefined" (23 occurrences)
14:36 - Alert: Error rate exceeded threshold
14:38 - Rollback to Deploy #46
14:39 - Error rate back to normal
```

Without this timeline, debugging is guesswork. With it, you know exactly which deploy caused the problem.

## The Mental Model

**Think of deployment as a conveyor belt, not a forklift.** A forklift picks up a heavy load, carries it somewhere, and puts it down. If the forklift breaks, nothing moves. A conveyor belt moves things continuously, one piece at a time. If one piece is defective, you pull it off and the belt keeps running. CI/CD is a conveyor belt: code flows from your editor to production continuously, and any single change can be inspected, tested, and removed without stopping the flow.

**Think of environments as dress rehearsals.** You wouldn't perform a play for a paying audience without a rehearsal. Development is practicing your lines at home. Preview/staging is the dress rehearsal (same stage, same costumes, small audience). Production is opening night. This means whenever you're unsure about a change, the answer is "test it in staging" not "hope it works in production."

**Think of rollbacks as an undo button that you set up before you need it.** The time to figure out your rollback strategy is before something breaks, not during an outage at midnight. This means whenever you deploy a new feature, you should already know: how do I undo this in under 5 minutes?

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Deployment | Putting your code on a server where users can access it. | Every time you ship a change. |
| CI (Continuous Integration) | Automatically testing every code change before it's merged. | PR checks, status badges. |
| CD (Continuous Deployment) | Automatically deploying code that passes tests. | When main is updated and the app auto-deploys. |
| Pipeline | The sequence of steps code goes through from push to production. | CI/CD configuration, workflow files. |
| Workflow | A GitHub Actions term for an automated pipeline defined in YAML. | `.github/workflows/` files. |
| Runner | The machine that executes your CI/CD pipeline. | GitHub Actions configuration (`runs-on`). |
| Artifact | A file produced by one step that another step needs (build output, test results). | Multi-step workflows. |
| Secret | A sensitive value (API key, token) stored encrypted in your CI/CD platform. | `${{ secrets.KEY }}` in GitHub Actions. |
| CDN | Content Delivery Network. Servers around the world that serve your static files from the nearest location. | Vercel, Netlify, Cloudflare, asset hosting. |
| Edge | Running code at CDN locations close to users, not in a central server. | Edge functions, edge middleware. |
| Serverless function | Code that runs on demand (no always-on server). Spins up per request. | Vercel API routes, AWS Lambda, Netlify Functions. |
| Cold start | The delay when a serverless function hasn't run recently and needs to initialize. | First request after idle period. |
| Preview deployment | A temporary deployment of a PR branch with its own URL. | Every PR on Vercel/Netlify. |
| Staging | A production-like environment for testing before deploying to real users. | Pre-production testing. |
| Rollback | Reverting production to a previous working version. | When a deploy breaks something. |
| Blue-green deploy | Running two identical environments and switching traffic between them. | Zero-downtime deployments. |
| Canary deploy | Rolling out a change to a small percentage of users first. | High-traffic apps, risky changes. |
| Feature flag | A toggle that enables/disables a feature without deploying new code. | Gradual rollouts, kill switches. |
| Health check | An endpoint (usually `/health`) that returns 200 if the service is working. | Load balancers, uptime monitors, orchestrators. |
| Smoke test | A quick test after deployment that verifies core functionality works. | Post-deploy verification scripts. |

## Common Patterns

### Pattern 1: Next.js on Vercel with Auto-Deploy

**When to use it:** You have a Next.js app and want zero-config deployment with preview URLs on every PR.

**How it works:** Connect your GitHub repo to Vercel. Every push to main deploys to production. Every PR gets a preview URL. Environment variables are set in the Vercel dashboard.

**Setup:**

```bash
# Install Vercel CLI (optional, for local preview)
pnpm add -g vercel

# Link your project
vercel link

# Deploy (usually not needed — auto-deploys from GitHub)
vercel --prod
```

Vercel dashboard configuration:
1. Import Git Repository
2. Select your repo
3. Framework: Next.js (auto-detected)
4. Environment Variables: add `DATABASE_URL`, etc.
5. Deploy

That's it. Every push to main auto-deploys. Every PR gets a preview. Rollback by clicking any previous deployment in the dashboard.

### Pattern 2: GitHub Actions Running Tests on Every PR

**When to use it:** You want every PR to pass linting, type checking, and tests before it can be merged.

**How it works:** A workflow file triggers on pull requests. It installs dependencies, runs checks, and reports pass/fail. GitHub blocks merging until the checks pass (if you enable branch protection).

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm type-check
      - run: pnpm test -- --coverage

      - name: Upload coverage
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

Enable branch protection in GitHub: Settings > Branches > Add rule > Require status checks > Select "quality."

### Pattern 3: Docker App on Railway/Fly.io

**When to use it:** You have a backend API or full-stack app that needs a persistent process (not serverless).

**How it works:** Push your code with a Dockerfile. Railway/Fly.io builds the image and runs it. Connect a managed database for persistence.

**Railway:**

```bash
# Install Railway CLI
pnpm add -g @railway/cli

# Login and link
railway login
railway link

# Deploy (auto-detects Dockerfile)
railway up

# Or just connect GitHub for auto-deploy
# Railway dashboard > New Project > Deploy from GitHub repo
```

**Fly.io:**

```bash
# Install Fly CLI
curl -L https://fly.io/install.sh | sh

# Login
fly auth login

# Launch (creates fly.toml config)
fly launch

# Deploy
fly deploy
```

Both platforms support: auto-deploy from GitHub, environment variables via dashboard or CLI, managed Postgres, custom domains, and horizontal scaling.

### Pattern 4: Monorepo Deployment with Multiple Services

**When to use it:** Your repo contains both a frontend and a backend (or multiple services) that deploy to different platforms.

**How it works:** Use path filters in GitHub Actions to only run workflows when relevant files change. Deploy each service to its own platform.

```yaml
# .github/workflows/deploy-frontend.yml
name: Deploy Frontend

on:
  push:
    branches: [main]
    paths:
      - "frontend/**"
      - "shared/**"          # Shared code used by frontend

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"
```

```yaml
# .github/workflows/deploy-backend.yml
name: Deploy Backend

on:
  push:
    branches: [main]
    paths:
      - "backend/**"
      - "shared/**"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --config backend/fly.toml
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

Frontend changes only trigger frontend deploy. Backend changes only trigger backend deploy. Changes to shared code trigger both.

### Pattern 5: Manual Approval Gates for Production

**When to use it:** You want CI to run automatically, but production deployment requires a human to approve it.

**How it works:** GitHub Actions environments can require reviewers. The workflow pauses at the deployment step and waits for approval.

```yaml
name: Deploy with Approval

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm test

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploying to staging..."
      # Deploy to staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production        # This environment requires approval
      url: https://myapp.com
    steps:
      - uses: actions/checkout@v4
      - run: echo "Deploying to production..."
      # Deploy to production
```

Configure in GitHub: Settings > Environments > "production" > Required reviewers. The workflow runs tests, deploys to staging automatically, then pauses. A designated reviewer clicks "Approve" in the Actions tab, and the production deploy proceeds.

## Mistakes Everyone Makes

### 1. No staging environment (testing in production)

**What people do wrong:** They deploy directly to production and hope it works. When it doesn't, real users see the errors.

**Why it seems right:** Setting up staging feels like extra work for a solo project. "I'll just test locally and deploy."

**What actually happens:** Local and production are never identical. Environment variables differ. Database schemas differ. API rate limits differ. The thing that works perfectly on localhost breaks in production because of a missing env var, a database migration that wasn't run, or a third-party API that behaves differently with production credentials.

**What to do instead:** Use preview deployments as your staging environment. On Vercel, every PR gets a preview URL automatically. Set preview environment variables to point at a staging database. Test there before merging. For backend services, create a staging environment on Railway or Fly.io (most platforms offer this at the project level). Total extra cost: usually $0.

### 2. Secrets hardcoded in GitHub Actions files

**What people do wrong:**

```yaml
# DON'T DO THIS
- run: curl -H "Authorization: Bearer sk-live-abc123" https://api.stripe.com/...
```

Or slightly less obviously:

```yaml
env:
  DATABASE_URL: postgres://user:realpassword@prod-db.example.com:5432/myapp
```

**Why it seems right:** The workflow file needs the values. Putting them right there makes it work.

**What actually happens:** The workflow file is committed to git. Everyone with repo access can see the secrets. If the repo is public, the entire internet can see them. Even in private repos, secrets in code tend to leak through forks, screenshots, or accidental public switches.

**What to do instead:** Use GitHub Secrets. Go to Settings > Secrets and variables > Actions > New repository secret. Reference them in workflows:

```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

GitHub automatically masks these values in logs. Even if a step tries to `echo` the secret, it shows `***` instead.

### 3. Not running tests before deploy

**What people do wrong:** Their CI/CD pipeline goes straight from push to deploy. No tests, no linting, no type checking.

**Why it seems right:** "Tests slow down the pipeline. I'll add them later. I just want to ship."

**What actually happens:** You ship a syntax error that crashes the app. Or a regression that breaks the login flow. Or a type error that only manifests with certain input. You find out when a user messages you. Or worse, when you check hours later and realize the app has been down.

**What to do instead:** At minimum, run these checks before any deploy:

```yaml
- run: pnpm lint          # Catches syntax errors, import issues
- run: pnpm type-check    # Catches type errors (TypeScript)
- run: pnpm test          # Catches regressions
- run: pnpm build         # Catches build-time errors
```

This adds 2-3 minutes to your pipeline. It saves you hours of production debugging. Make it a non-negotiable step.

### 4. No rollback plan

**What people do wrong:** They deploy and assume it'll work. When it doesn't, they panic, try to fix it live, push a hasty fix that introduces another bug, and make things worse.

**Why it seems right:** "It worked in staging, it'll work in production."

**What actually happens:** Production has data, traffic, and edge cases that staging doesn't. A migration that works on an empty database fails on a table with 100,000 rows. A new feature works for 90% of users but crashes for users with certain account settings. You need to get back to a working state fast.

**What to do instead:** Before every deploy, know the answer to: "How do I undo this in under 5 minutes?" For Vercel/Netlify, the answer is "click the previous deployment and promote it." For Railway/Fly.io, the answer is "roll back to the previous release." For database migrations, the answer is "have a down migration ready." Practice the rollback before you need it.

### 5. Ignoring build times until they hit 10+ minutes

**What people do wrong:** They don't think about build performance until their pipeline takes 15 minutes. By then, developers push less often (because waiting is painful), PRs get bigger (because changes are batched), and feedback loops slow to a crawl.

**Why it seems right:** Build times creep up slowly. 3 minutes feels fine. Then 5. Then 8. By the time it's painful, there's no single cause to fix.

**What actually happens:** Slow pipelines kill shipping velocity. Developers start skipping CI ("I'll push straight to main, the tests take too long"). Preview deployments become useless because they take 15 minutes to build. The entire CI/CD system, which exists to make shipping fast, becomes the bottleneck.

**What to do instead:** Track build times from the start. Set a budget: total pipeline should be under 5 minutes. When it approaches the budget, investigate:

- **Cache dependencies.** Use `actions/cache` or the built-in cache options in `actions/setup-node`. Dependencies should only install when the lockfile changes.
- **Run independent jobs in parallel.** Linting, type checking, and tests can all run at the same time.
- **Use path filters.** Don't run the entire pipeline when only docs changed.
- **Consider splitting test suites.** Unit tests run on every PR. Integration tests run on merge to main.

### 6. Not using branch protection rules

**What people do wrong:** Anyone can push directly to main, bypassing all CI checks.

**Why it seems right:** "I'm the only developer. Branch protection is for teams."

**What actually happens:** You push a quick fix directly to main at 11 PM. You skip tests because "it's just a one-line change." The one-line change breaks the build. Production is down. The CI/CD pipeline that's supposed to protect you was bypassed because nothing enforced it.

**What to do instead:** Enable branch protection on main, even for solo projects. Require status checks to pass. Require PRs (even if you're merging your own). It takes 30 seconds to set up and prevents the most common "I was in a hurry" disasters. Settings > Branches > Add rule > require status checks.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Every deploy to production must pass automated checks first.** No exceptions, no "just this once." The one time you skip it is the time it breaks.

2. **Keep the pipeline under 5 minutes.** Cache dependencies, parallelize jobs, use path filters. Slow pipelines get bypassed.

3. **Every environment variable should exist in exactly one place per environment.** Don't duplicate secrets across platforms. Don't have env vars in code "just as defaults."

4. **Preview deployments should mirror production as closely as possible.** Same build process, same environment variable structure (different values), same database schema. The further preview drifts from production, the less useful it is.

5. **Make rollbacks a one-step operation.** Whether that's clicking a button in Vercel or running a single CLI command, you should be able to revert to the previous working version without thinking.

6. **Monitor after every deploy, not just when something feels wrong.** Check error rates, response times, and key user flows for 15 minutes after each deploy. Automated alerts are better, but manually checking is better than nothing.

7. **Treat CI/CD configuration as code that deserves review.** Workflow files go in PRs, get reviewed, and are tested in preview before landing on main. A broken CI/CD pipeline is worse than a broken feature.

8. **Don't deploy on Fridays.** Or before holidays. Or before you go to sleep. Deploy when you're available to respond if something goes wrong. If you have solid monitoring, feature flags, and instant rollbacks, this rule relaxes, but respect it until then.

## The "Just Tell Me What to Do" Quickstart

You'll set up a Next.js app on Vercel with a GitHub Actions CI pipeline. Total time: under 10 minutes.

**Prerequisites:** GitHub account, Vercel account (free tier), Node.js installed, pnpm installed.

**Step 1: Create a Next.js app**

```bash
pnpm create next-app@latest my-deployed-app --typescript --tailwind --app --src-dir --eslint
cd my-deployed-app
```

**Step 2: Add a test script**

```bash
pnpm add -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/jest-dom
```

Create `vitest.config.ts`:

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test-setup.ts"],
  },
});
```

Create `src/test-setup.ts`:

```typescript
import "@testing-library/jest-dom/vitest";
```

Create `src/app/page.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import Home from "./page";

describe("Home", () => {
  it("renders without crashing", () => {
    render(<Home />);
    expect(document.body).toBeTruthy();
  });
});
```

Add scripts to `package.json`:

```json
{
  "scripts": {
    "test": "vitest run",
    "type-check": "tsc --noEmit"
  }
}
```

Verify it works:

```bash
pnpm test
pnpm type-check
pnpm lint
```

**Step 3: Create the GitHub Actions workflow**

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Type check
        run: pnpm type-check

      - name: Test
        run: pnpm test

      - name: Build
        run: pnpm build
```

**Step 4: Push to GitHub**

```bash
git init
git add -A
git commit -m "feat: initial setup with CI pipeline"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/my-deployed-app.git
git push -u origin main
```

**Step 5: Connect to Vercel**

1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your GitHub repository
3. Framework: Next.js (auto-detected)
4. Click Deploy

Your app is live. The URL is shown in the dashboard.

**Step 6: Test the pipeline**

```bash
git checkout -b test-ci
echo "// test change" >> src/app/page.tsx
git add -A
git commit -m "test: verify CI pipeline"
git push -u origin test-ci
```

Open a PR on GitHub. You'll see: the CI workflow running checks and Vercel deploying a preview. Both post their status on the PR.

**Step 7: Clean up and merge**

If checks pass, merge the PR. Vercel auto-deploys the updated main branch. Your CI/CD pipeline is working.

## How to Prompt AI Tools About This

### Context to provide

When asking Claude, Cursor, or Copilot about deployment and CI/CD, always specify:
- Your framework and version ("Next.js 15," "Express with TypeScript")
- Your hosting platform ("Vercel," "Railway," "Fly.io")
- Whether you need CI, CD, or both
- Your package manager ("pnpm," "npm")
- Any services your app depends on ("Postgres," "Redis," "Stripe")

### Example prompts that produce good results

**GitHub Actions workflow:**
> "Write a GitHub Actions workflow for a Next.js 15 app using pnpm. Run lint, type-check, and vitest on every PR. Cache pnpm dependencies. Run jobs in parallel where possible. Keep total pipeline under 3 minutes."

**Deployment configuration:**
> "I have a Next.js app that uses Supabase for the database and Stripe for payments. Write the environment variable setup for Vercel (production and preview environments). Which variables need NEXT_PUBLIC_ prefix and which don't?"

**Docker deployment:**
> "Write a Dockerfile and fly.toml for deploying a Fastify API (TypeScript) to Fly.io. Multi-stage build, non-root user, health check endpoint. The app connects to a Fly Postgres database."

**Debugging deploy failures:**
> "My Vercel build fails with 'Module not found: Can't resolve @/lib/utils'. It works locally. Here's my tsconfig.json: [paste]. What's different about the Vercel build environment?"

**Monorepo setup:**
> "I have a monorepo with frontend/ (Next.js) and backend/ (Fastify). Write GitHub Actions workflows that only deploy the service that changed. Frontend goes to Vercel, backend goes to Railway."

### What to watch out for in AI-generated CI/CD code

- **Outdated action versions.** AI might generate `actions/checkout@v3` when `v4` is current. Always check for the latest major version.
- **Missing pnpm setup.** AI often forgets the `pnpm/action-setup` step and uses npm instead. If your project uses pnpm, add the setup step explicitly.
- **Overly permissive triggers.** AI tends to generate `on: [push, pull_request]` without branch filters, which runs the workflow on every push to every branch. Narrow it to `branches: [main]`.
- **No dependency caching.** AI sometimes skips the `cache: "pnpm"` option in `actions/setup-node`, which makes every run reinstall from scratch.
- **Hardcoded values that should be secrets.** Check every URL, token, and API key in the generated workflow. If it's sensitive, it should be `${{ secrets.NAME }}`.
- **Incorrect working directory for monorepos.** AI may not add `working-directory: frontend/` to steps when your code isn't in the repo root.

### Key terms to use in prompts

Using these terms produces noticeably better output: "GitHub Actions workflow," "branch protection," "preview deployment," "environment secrets," "frozen-lockfile," "path filter," "required status checks," "pnpm cache," "parallel jobs."

## Ship It: Build This

Set up a complete deployment pipeline for a Next.js app with automated testing, preview deployments, and deploy notifications.

**What you're building:** A Next.js application deployed on Vercel with a GitHub Actions workflow that runs linting, type checking, and tests on every PR, auto-deploys to production when merged to main, and sends a notification to Discord (or Slack) whenever a production deploy succeeds or fails.

**Why this project specifically:** It covers the entire CI/CD lifecycle. You'll set up automated quality checks (CI), automatic deployment (CD), preview environments (for PR review), and monitoring notifications (so you know when things ship or break). This is the same pipeline used by professional teams, just without the enterprise complexity.

**Rough architecture:**

- Next.js app with at least one page and one test
- Vercel for hosting (auto-deploy from main, preview deploys on PRs)
- GitHub Actions for CI: lint, type-check, test, build on every PR
- GitHub Actions for deploy notification: triggers after Vercel deploy, sends Discord/Slack webhook
- Branch protection on main: require CI checks to pass before merge
- Environment variables split correctly between Vercel environments (production, preview)
- A `/health` API route that returns 200 (for uptime monitoring)
- UptimeRobot configured to ping the health endpoint every 5 minutes

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn blue-green and canary deployments** when you need zero-downtime deploys or want to test changes on a small percentage of traffic before full rollout.
- **Learn infrastructure as code (Terraform, Pulumi)** when you're managing cloud resources (databases, queues, storage buckets) and want them version-controlled and reproducible.
- **Learn container orchestration (Kubernetes)** when you're running many services at scale and need automated scaling, self-healing, and rolling deploys across a cluster.
- **Learn database migration strategies** when your deploys include schema changes and you need to handle them without downtime or data loss (expand-and-contract pattern, zero-downtime migrations).
- **Learn observability (OpenTelemetry, distributed tracing)** when your system has multiple services and you need to trace a request across all of them to find where it's slow or failing.
