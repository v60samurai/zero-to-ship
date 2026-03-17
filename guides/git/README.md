# Git: Zero to Ship

> Git saves snapshots of your code so you can go back in time, work on features without breaking things, and collaborate without overwriting each other's work.

## Why Should You Care?

You're building with Cursor, Claude Code, or Lovable. The AI writes most of your code. Why bother understanding Git?

Because Git isn't about writing code. It's about controlling what happens to code after it's written. You need Git the moment you care about any of these: going back to yesterday's version because today's AI-generated refactor broke everything, working on a new feature while keeping the live app stable, collaborating with anyone (even future you), or deploying code to production safely.

Here's the concrete scenario. You prompt Claude Code to refactor your dashboard component. It rewrites 200 lines. The app breaks. Without Git, you're staring at a broken file trying to remember what it looked like 10 minutes ago. With Git, you run one command and you're back to the working version. Then you try the refactor again on a separate branch where breaking things is free.

AI tools make you faster at writing code. Git makes you safer while moving fast. The faster you ship, the more you need a safety net.

## The 30-Second Version

Git tracks every change you make to your code as a series of snapshots called "commits." You choose when to take a snapshot. You can create parallel versions of your project called "branches" to try things safely. When a branch works, you merge it back. When it doesn't, you delete it. Nothing on the main version was ever at risk.

## The Real Explanation (ELI5)

Imagine you're writing a school essay in a notebook. Every few paragraphs, you photocopy the entire notebook and put the copy in a filing cabinet with today's date on it. If you mess up a paragraph tomorrow, you pull out yesterday's copy and start from there.

That's committing. The notebook is your code. The filing cabinet is your repository. Each photocopy is a commit.

Now imagine you want to try a completely different ending for your essay, but you're not sure it'll work. So you photocopy the notebook, label it "experimental ending," and write in that copy instead. If the new ending is great, you rip out the last pages of your original notebook and replace them with the new ending. If it's terrible, you throw the experimental copy away. Your original notebook was never at risk.

That's branching and merging. The experimental copy is a branch. Replacing the pages is a merge. Throwing it away is deleting the branch.

Now replace the notebook with your project folder, the filing cabinet with a `.git` folder on your computer, and the photocopies with snapshots that Git takes when you run `git commit`. The process is identical.

One thing the analogy doesn't capture: Git also lets you sync your filing cabinet with one that lives on the internet (GitHub). That's how teammates access the same code, and how your code gets deployed to servers.

## How It Actually Works

There are three places your code can be at any moment:

```
┌─────────────────┐     git add     ┌─────────────────┐    git commit    ┌─────────────────┐
│  Working         │ ──────────────► │  Staging Area    │ ──────────────► │  Repository      │
│  Directory       │                 │  (the box)       │                 │  (the vault)     │
│                  │ ◄────────────── │                  │                 │                  │
│  Your actual     │   git restore   │  Changes you've  │                 │  Permanent       │
│  files on disk   │   --staged      │  picked for the  │                 │  snapshots of    │
│                  │                 │  next snapshot   │                 │  your project    │
└─────────────────┘                 └─────────────────┘                 └─────────────────┘
                                                                               │
                                                                          git push
                                                                               │
                                                                               ▼
                                                                        ┌─────────────────┐
                                                                        │  GitHub          │
                                                                        │  (remote)        │
                                                                        │                  │
                                                                        │  The cloud copy  │
                                                                        │  everyone shares │
                                                                        └─────────────────┘
```

**What happens when you save a feature:**

1. You edit files in your working directory (just normal coding).
2. You run `git add` to pick which changes go into the next snapshot. This moves them to the staging area.
3. You run `git commit` to take the snapshot. This saves them permanently in your local repository.
4. You run `git push` to upload that snapshot to GitHub so others can see it.

**Under the hood:**

Git doesn't copy your entire project every time you commit. It stores only what changed (called a "diff") and a pointer to the previous commit. This chain of pointers is your history. Every commit has a unique ID (a hash like `a3f9d12`) so Git can reference any snapshot instantly.

**Why it's designed this way:**

The staging area feels like an unnecessary step. Why not just commit everything? Because sometimes you changed five files but only three of them are related to the feature you're shipping. Staging lets you group related changes into one clean commit and leave the rest for later. It's the difference between a commit that says "feat: add dark mode" and one that says "add dark mode and also fix a typo and also experiment with new font."

## The Mental Model

**Think of commits as save points in a video game.** This means whenever you reach a working state (feature works, bug is fixed, refactor is done), you save. If the next thing you try goes wrong, you load the save point instead of restarting the whole level.

**Think of branches as parallel universes.** This means whenever you want to try something risky, you branch off into a parallel universe. The main universe is untouched. If the experiment works, the two universes merge. If it fails, the parallel universe just disappears. This is why you never code directly on `main`. That branch is the "real" universe.

**Think of `git push` as uploading and `git pull` as downloading.** This means every morning you download the latest version before you start working (`git pull`), and every time you finish something, you upload it (`git push`). If you forget to download first, you'll be editing an outdated version, and merging gets messy.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Repository (repo) | A project folder that Git is tracking. Contains all your files plus the hidden `.git` folder that stores history. | Every Git project is a repo. You "clone a repo" or "create a repo." |
| Commit | A snapshot of your project at one moment in time, with a message describing what changed. | Every time you save progress. `git commit -m "feat: add login page"` |
| Staging area | A holding zone where you pick which changes go into the next commit. | When you run `git add`. Think of it as packing a box before sealing it. |
| Branch | A parallel version of your codebase. Changes on a branch don't affect other branches until you merge. | Every feature or bug fix gets its own branch. |
| Main (or master) | The default branch. Usually represents the live, stable version of your code. | The branch you merge finished work into. The one running in production. |
| Merge | Combining changes from one branch into another. | When a feature is done and reviewed, you merge it into main. |
| Pull Request (PR) | A request on GitHub to merge your branch into another branch. Where code review happens. | After pushing your branch, you open a PR on GitHub. |
| Remote | A copy of the repo that lives somewhere else, usually GitHub. The default remote is called "origin." | When you push or pull. `origin` is the name for your GitHub copy. |
| Clone | Download an entire repo (with all its history) from GitHub to your computer. | First time setting up a project someone else started. `git clone [url]` |
| Hash | The unique ID for a commit, like `a3f9d12`. Git generates it automatically. | When you need to reference a specific commit for `revert`, `show`, or `reset`. |
| HEAD | A pointer to the commit you're currently looking at. Usually the latest commit on your current branch. | In commands like `HEAD~1` (one commit before the current one). |
| Diff | The specific lines that changed between two versions of a file. | When reviewing changes. `git diff` shows what you've modified. |
| Merge conflict | When two people edit the same lines in the same file and Git can't decide which version to keep. | When pulling or merging and Git finds overlapping edits. |
| .gitignore | A file that tells Git which files to never track (secrets, dependencies, build output). | Set up once at the start of a project. Prevents accidents. |
| Stash | A temporary hiding spot for half-done work you're not ready to commit. | When you need to switch branches urgently without committing incomplete work. |
| Tag | A permanent label on a specific commit, usually used to mark releases. | When shipping a version. `git tag v1.0.0` |
| Rebase | Replaying your commits on top of someone else's changes to keep history linear. | In `git pull --rebase`. Cleaner than merge commits for daily syncing. |

## Common Patterns

### Pattern 1: Solo Developer Workflow

**When to use it:** You're building alone. No teammates, no code review. Just you and your AI tools.

**How it works:** You work directly on `main` for small changes and create branches for anything that might break things. Commits are your save points. Tags mark versions you deploy.

```bash
# Small fix, commit directly
git add .
git commit -m "fix: correct API endpoint typo"
git push origin main

# Bigger feature, use a branch
git checkout -b feat/payment-page
# ... build the feature ...
git add .
git commit -m "feat: add Stripe payment page"
git checkout main
git merge feat/payment-page
git push origin main
git branch -d feat/payment-page
```

### Pattern 2: Team Feature Branch Workflow

**When to use it:** You're working with at least one other person. The standard for most teams.

**How it works:** Nobody touches `main` directly. Every change goes through a branch and a Pull Request. PRs get reviewed before merging. This is the workflow described in the production section below.

```bash
git checkout -b feat/user-dashboard
# ... build, commit multiple times ...
git push origin feat/user-dashboard
# Open PR on GitHub → get review → merge → delete branch
```

### Pattern 3: Hotfix (Emergency Production Fix)

**When to use it:** Something is broken in production right now. You need to fix it without shipping half-finished feature work.

**How it works:** Branch off `main` (not your feature branch), fix the bug, push, merge immediately.

```bash
git stash                          # hide your current work
git checkout main
git pull --rebase origin main
git checkout -b fix/checkout-crash
# ... fix the one bug ...
git add .
git commit -m "fix: null check on checkout total"
git push origin fix/checkout-crash
# Open PR, get fast review, merge
git checkout your-feature-branch
git stash pop                      # restore your work
```

### Pattern 4: "I Need to Start Over on This Branch"

**When to use it:** Your branch is a mess. AI-generated refactors went sideways. Easier to restart than untangle.

**How it works:** Delete the branch and recreate it from a clean `main`.

```bash
git checkout main
git pull --rebase origin main
git branch -D feat/broken-experiment    # -D force deletes
git checkout -b feat/broken-experiment  # fresh start from main
```

### Pattern 5: Open Source Contribution (Fork and PR)

**When to use it:** You want to contribute to someone else's project on GitHub.

**How it works:** You "fork" their repo (creates your own copy on GitHub), clone your fork, make changes on a branch, push to your fork, then open a PR from your fork to the original repo.

```bash
# After forking on GitHub:
git clone https://github.com/YOU/their-project.git
cd their-project
git checkout -b fix/typo-in-docs
# ... make changes ...
git add .
git commit -m "fix: correct typo in installation docs"
git push origin fix/typo-in-docs
# Open PR on GitHub from your fork to the original repo
```

## Mistakes Everyone Makes

### 1. Committing .env files with secrets

**What people do wrong:** They commit their `.env` or `.env.local` file that contains API keys, database URLs, and secret tokens.

**Why it seems right:** "It's just a file in my project. Git should track all my files."

**What actually happens:** The secrets are now in Git history permanently. Even if you delete the file in the next commit, anyone can see the old version. Bots actively scan GitHub for exposed secrets. If you push AWS keys, expect unauthorized charges within hours.

**What to do instead:** Create a `.gitignore` file before your first commit with `.env` and `.env.local` listed. If you already committed secrets, assume they're compromised. Rotate every key immediately. Then use `git filter-branch` or BFG Repo-Cleaner to remove them from history.

```
# .gitignore (create this FIRST, before any commits)
.env
.env.local
node_modules/
.next/
dist/
.DS_Store
*.log
```

### 2. Running `git add .` without checking what changed

**What people do wrong:** They blindly stage everything without running `git status` first.

**Why it seems right:** "I just want to commit my work. Stage everything."

**What actually happens:** You accidentally commit debug logs, console.log statements, temp files, `.DS_Store`, or half-written code in other files you forgot you touched. Your commit becomes a grab bag instead of a clean change.

**What to do instead:** Always run `git status` before `git add .`. Scan the list. If something doesn't belong in this commit, stage files individually with `git add [specific-file]`.

### 3. Vague commit messages

**What people do wrong:** Messages like "fixed stuff," "updates," "wip," or "asdfgh."

**Why it seems right:** "I'll remember what this was. Commit messages don't matter."

**What actually happens:** Three weeks later, the checkout page breaks. You run `git log --oneline` and see:

```
a3f9d12 updates
b8e2c45 fixed stuff
9f1a3b6 wip
c2d4e67 more changes
```

You have no idea which commit to revert. You end up reading diffs on every single one.

**What to do instead:** Use the `type: description` format. One logical change per commit.

```
feat: add CSV export to reports
fix: prevent double-submit on checkout form
chore: upgrade Next.js to 15.2
refactor: extract auth logic into hook
```

### 4. Using `git reset --hard` on shared commits

**What people do wrong:** They run `git reset --hard HEAD~3` to undo commits that are already pushed to GitHub.

**Why it seems right:** "I want to undo these commits. Reset sounds like the right command."

**What actually happens:** You rewrite history on your machine, but GitHub and your teammates still have the old history. When anyone pulls, they get conflicts that are nearly impossible to resolve cleanly. The error message is something like:

```
! [rejected] main -> main (non-fast-forward)
hint: Updates were rejected because the tip of your current branch is behind
```

**What to do instead:** For commits already on GitHub, use `git revert [hash]`. It creates a new commit that undoes the changes without rewriting history. `git reset` is only safe for commits that haven't been pushed yet.

### 5. Working on main instead of a branch

**What people do wrong:** They start coding a feature directly on `main`.

**Why it seems right:** "Branches are extra steps. I'll just commit to main."

**What actually happens:** Halfway through the feature, an urgent bug report comes in. Your `main` branch now has half-finished feature code mixed with the production code. You can't deploy the bug fix without also deploying the incomplete feature. You're stuck.

**What to do instead:** Before writing any code, run `git checkout -b feat/your-feature`. It takes 2 seconds. Now `main` is always clean and deployable.

### 6. Forgetting to pull before starting work

**What people do wrong:** They start coding in the morning without running `git pull` first.

**Why it seems right:** "I was the last one working yesterday. Nothing changed."

**What actually happens:** Your teammate merged a PR last night. You're now editing an outdated version of the codebase. When you try to push, you get merge conflicts that could have been avoided entirely.

**What to do instead:** Make `git pull --rebase origin main` the first command you run every morning. It takes 3 seconds and prevents hours of conflict resolution.

### 7. Giant commits with unrelated changes

**What people do wrong:** They code for 6 hours, then do one massive `git add . && git commit -m "built the feature"`.

**Why it seems right:** "I was in the zone. I'll commit when I'm done."

**What actually happens:** The commit touches 30 files across 4 different concerns. If one part introduces a bug, you can't revert just that part. You have to revert the entire 6 hours of work.

**What to do instead:** Commit every time you complete one logical unit of work. Finished the API route? Commit. Finished the UI for it? Commit. Fixed a bug you noticed along the way? Separate commit.

## Best Practices (The Rules That Prevent 90% of Problems)

These aren't Git-specific. They're codebase maintenance habits that keep projects healthy as they grow. Git is just the tool that enforces them.

**1. One branch per concern, one commit per change.**
Mixing unrelated work in the same branch or commit makes it impossible to undo one thing without undoing everything.

**2. Never commit secrets, credentials, or API keys.**
Treat every file in your repo as public. If a value would be dangerous in a stranger's hands, it goes in `.env` (which is in `.gitignore`), not in your code.

**3. Keep `main` deployable at all times.**
If someone clones your repo and runs the app from `main`, it should work. Broken code lives on branches until it's fixed.

**4. Pull before you push, every time.**
Syncing before you start work prevents most merge conflicts. Three seconds of habit saves hours of untangling.

**5. Write commit messages for the person debugging at 2 AM.**
That person is usually future you. "fix: prevent double charge on retry" is useful. "fixed bug" is not.

**6. Delete branches after merging.**
Stale branches accumulate fast. If it's merged, it's done. Delete it locally and on GitHub. You can always find the code in the merge history.

**7. Review your own PR before asking others to.**
Read through the diff on GitHub before requesting review. You'll catch leftover console.logs, debug code, and accidental file changes that waste a reviewer's time.

**8. Tag releases with semantic versions.**
When you deploy, tag it (`v1.2.0`). When something breaks in production, you can instantly see what version is running and roll back to the previous tag instead of guessing which commit was the last stable one.

**9. Keep your `.gitignore` comprehensive from day one.**
Add OS files (`.DS_Store`), dependency folders (`node_modules/`), build output (`dist/`, `.next/`), and environment files before your first commit. Cleaning these out of history after the fact is painful.

**10. Small PRs ship faster and break less.**
A 50-line PR gets reviewed in 10 minutes. A 500-line PR sits in the queue for days and reviewers skim instead of reading. If a feature is big, split it into multiple PRs that each do one thing.

## The "Just Tell Me What to Do" Quickstart

This gets you from zero to a working Git repo pushed to GitHub in under 5 minutes. You'll need a GitHub account and a terminal.

**Step 1: Set up Git (one-time only)**

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

**Step 2: Create a project and initialize Git**

```bash
mkdir my-project && cd my-project
git init
```

**Step 3: Create a .gitignore**

```bash
echo "node_modules/\n.env\n.env.local\n.next/\ndist/\n.DS_Store\n*.log" > .gitignore
```

**Step 4: Create some files and make your first commit**

```bash
echo "# My Project" > README.md
git add .
git commit -m "chore: initial project setup"
```

**Step 5: Create a repo on GitHub**

Go to github.com/new. Create a repo with the same name. Do NOT initialize with a README (you already have one). Copy the repo URL.

**Step 6: Connect and push**

```bash
git remote add origin https://github.com/yourusername/my-project.git
git branch -M main
git push -u origin main
```

**Step 7: Make a change on a branch (the daily workflow)**

```bash
git checkout -b feat/hello-world
echo "console.log('hello world');" > index.js
git add .
git commit -m "feat: add hello world script"
git push origin feat/hello-world
```

Now go to GitHub. You'll see a prompt to create a Pull Request. Click it, review your changes, and merge.

**Step 8: Sync and clean up**

```bash
git checkout main
git pull --rebase origin main
git branch -d feat/hello-world
```

You now have the complete loop. Everything else in Git is a variation of these steps.

## How to Prompt AI Tools About This

Git is one of the topics where AI tools are genuinely useful because the commands have consistent patterns but tricky edge cases. Here's how to get good results.

### Context to include in your prompts

Always tell the AI tool: what branch you're on, what you're trying to achieve (not what command you think you need), and paste the exact error message if something went wrong. Git errors are specific enough that AI can diagnose them from the message alone.

### Prompts that produce good results

**When you're stuck:**
```
I'm on branch feat/user-auth. I committed changes but haven't pushed yet.
I want to undo the last commit but keep the changes so I can split them
into two smaller commits. What's the command?
```

**When you have a merge conflict:**
```
I'm merging feat/dark-mode into main and I got a conflict in
src/components/Header.tsx. Here's the conflict:

[paste the conflict markers]

The intent of my branch was to add a theme toggle button.
The main branch updated the navigation links.
Both changes should exist in the final version.
Help me resolve this.
```

**When you need a workflow:**
```
I'm a solo developer deploying a Next.js app to Vercel.
Set up a Git workflow for me that includes:
- Branch naming convention
- When to tag releases
- How to handle hotfixes
Keep it simple. I don't need CI/CD yet.
```

### What to watch out for in AI-generated Git advice

AI tools sometimes suggest `git push --force` without warning you it rewrites history. If you see `--force` in a suggestion, pause and check if you're rewriting commits that others have already pulled. If so, ask the AI for an alternative that uses `git revert` instead.

AI also tends to suggest complex solutions for simple problems. If the suggestion involves `git cherry-pick`, `git rebase -i`, or `git reflog` and you're a beginner, ask: "Is there a simpler way to do this?" Often there is.

### Key terms that improve prompt quality

Use "undo last commit (keep changes)" instead of "undo commit." Use "merge conflict resolution" instead of "Git problem." Use "rewrite commit message" instead of "change message." The more specific your terminology, the more targeted the answer.

## Ship It: Build This

**Build a personal project with a clean Git history and deploy it.**

Take any small project (a portfolio site, a to-do app, a landing page) and build it using the full Git workflow from this guide. The goal isn't the project itself. The goal is practicing the workflow until branches, commits, and PRs feel automatic.

**Why this project:** It exercises every concept in the guide. Initializing a repo, branching, committing with good messages, pushing, creating PRs (even if you're merging your own), tagging a release, and deploying from `main`. If you can do all of this once without referring back to the guide, you have Git down.

**Rough plan:**

- Initialize the repo and push to GitHub with a `.gitignore` and README
- Build feature 1 on `feat/feature-one` branch, open a PR, merge it
- Build feature 2 on a separate branch while feature 1 is on main
- Intentionally create a merge conflict between the two branches and resolve it
- Tag the final version as `v1.0.0`
- Connect the repo to Vercel/Netlify so `main` auto-deploys

**Estimated time:** 2 to 3 hours for a vibe coder using AI tools (most of that time is the project itself, not Git).

## Go Deeper

- **Learn interactive rebase (`git rebase -i`)** when you need to clean up messy commit history before opening a PR. It lets you squash, reorder, and rename commits.
- **Learn Git hooks** when you want to automatically run linters or tests before every commit. Useful once your project is big enough to have a test suite.
- **Learn `git bisect`** when you have a bug and you know it worked two weeks ago but you don't know which commit broke it. Bisect does a binary search through your history to find the exact commit.
- **Learn `git worktree`** when you're tired of stashing. It lets you have two branches checked out simultaneously in different folders.
- **Learn GitHub Actions** when you want tests to run automatically on every PR. It's the next step after mastering Git itself.

## Quick Reference (Cheat Sheet)

```bash
# Setup
git init                                  # start tracking a folder
git remote add origin [url]               # connect to GitHub
git clone [url]                           # download a repo

# Daily loop
git status                                # see what changed
git add .                                 # stage everything
git add [file]                            # stage one file
git commit -m "type: description"         # take a snapshot
git push origin [branch]                  # upload to GitHub
git pull --rebase origin main             # download latest changes

# Branches
git checkout -b [branch-name]             # create and switch to a branch
git checkout [branch-name]                # switch to existing branch
git branch                                # list all branches
git branch -d [branch-name]              # delete a branch
git merge [branch-name]                   # merge a branch into current branch

# Undoing things
git restore [file]                        # undo uncommitted changes
git restore --staged [file]               # unstage a file
git reset --soft HEAD~1                   # undo last commit, keep changes
git revert [hash]                         # undo a pushed commit safely

# Stash
git stash                                 # hide current changes temporarily
git stash pop                             # bring them back
git stash list                            # see all stashes

# History
git log --oneline                         # compact history
git diff                                  # see what changed since last commit
git show [hash]                           # see what one commit changed

# Tags
git tag v1.0.0                            # label current commit
git push origin v1.0.0                    # push tag to GitHub
```
