---
name: git-basics
description: 'Git reference for everyday operations: pull, push, and checkout.'
---

# Git Basics Skill

## Skill Metadata
- **Name**: git-basics
- **Domain**: Git – pull, push, checkout
- **Purpose**: Quick reference for the most common Git operations used day-to-day

## When to Use This Skill

### Auto-Trigger Conditions (file patterns):
- `**/.github/**` - GitHub configuration or workflow files
- `**/pom.xml` - Maven project files (version updates require Git operations)

### Manual Trigger Keywords:
- "git pull"
- "git push"
- "git checkout"
- "fetch changes"
- "switch branch"
- "update branch"
- "push changes"

### Use During:
- Syncing your local branch with the remote
- Switching between feature/fix branches
- Publishing local commits to the remote repository

---

## Commands

### git pull

Fetch changes from the remote and merge them into the current branch.

```bash
# Pull latest changes from the tracked remote branch
git pull

# Pull from a specific remote and branch
git pull origin main

# Pull with rebase instead of merge (keeps history linear)
git pull --rebase origin main
```

> **Tip:** Use `git pull --rebase` to avoid unnecessary merge commits when working on a shared branch.

---

### git push

Upload local commits to the remote repository.

```bash
# Push current branch to its tracked remote branch
git push

# Push to a specific remote and branch
git push origin feature/my-branch

# Push a new local branch and set the upstream tracking
git push --set-upstream origin feature/my-branch
# Shorthand:
git push -u origin feature/my-branch

# Force-push after a rebase (use with caution!)
git push --force-with-lease origin feature/my-branch
```

> **Warning:** Prefer `--force-with-lease` over `--force`. It refuses to overwrite commits
> that others have pushed since your last fetch, preventing accidental data loss.

---

### git checkout

Switch branches or restore working-tree files.

```bash
# Switch to an existing branch
git checkout main

# Create and switch to a new branch from the current HEAD
git checkout -b feature/my-new-branch

# Create a new branch tracking a specific remote branch
git checkout -b feature/my-branch origin/feature/my-branch

# Discard local changes to a specific file (restore from HEAD)
git checkout -- path/to/file.txt
```

> **Tip:** In Git 2.23+, `git switch` and `git restore` are the modern alternatives:
> - `git switch main` → switch to an existing branch
> - `git switch -c feature/my-branch` → create and switch
> - `git restore path/to/file.txt` → discard local changes

---

## Common Workflows

### Start work on a new feature
```bash
git checkout main
git pull origin main
git checkout -b feature/my-feature
```

### Sync a feature branch with main before creating a PR
```bash
git checkout main
git pull origin main
git checkout feature/my-feature
git pull --rebase origin main
git push --force-with-lease origin feature/my-feature
```

### Push a finished feature branch and open a PR
```bash
git push -u origin feature/my-feature
gh pr create --title "feat: my feature" --base main
```

---

## Safety Rules
- Always run `git status` before pulling or pushing to understand the state of your working tree.
- Never force-push to protected branches (e.g., `main`, `develop`, `master`).
- Prefer `--rebase` over merge pulls to keep history readable.
- Use `--force-with-lease` instead of `--force` when a rebase makes a force-push necessary.
