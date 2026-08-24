# Git Practical Guide & Workflow Cheatsheet

This guide documents the Git workflows, commands, and best practices discussed for repository management, synchronization, multi-version development (v1.0 vs v2.0), Worktrees, and cherry-picking bug fixes without leaking unfinished features.

---

## 1. Remote Synchronization & Updating from GitHub

### Check Repository State
```powershell
# Check current branch and uncommitted/untracked files
git status

# Check configured remote repository URLs
git remote -v
```

### Fetch and Pull Latest Remote Changes
```powershell
# 1. Fetch all branches and prune deleted remote branches
git fetch --all --prune

# 2. Fast-forward pull local master to match origin/master
git pull --ff-only
```

---

## 2. Staging, Committing & Pushing Changes

### Step-by-Step
```powershell
# Stage a specific file
git add path/to/file.ext

# Or stage all changed & new files
git add .

# Commit with a meaningful message
git commit -m "docs: add Traditional Chinese user guide"

# Push to GitHub
git push origin master
```

### Quick One-Liner
```powershell
git add . && git commit -m "Your commit message" && git push origin master
```

---

## 3. Git Worktrees (Multi-Version & Parallel Development)

### What is a Worktree?
A Git Worktree lets you check out multiple branches simultaneously into completely separate folders on your disk while sharing the same `.git` repository, history, and tags.

### Key Use Cases
1. **Developing v2.0 alongside v1.0**: Work on `v2.0` in `c:\ai\mds-v2` while `c:\ai\mds` remains on stable `master`.
2. **Urgent Hotfixes**: Fix production bugs on `master` without stashing or disrupting active feature work.
3. **AI Pair Programming / Subagents**: Give an AI agent its own worktree directory to prevent conflicting edits in your primary IDE workspace.
4. **Side-by-Side Dev Servers**: Run both `v1.0` (port 3000) and `v2.0` (port 3001) simultaneously to visually test regressions.

### Worktree Commands
```powershell
# 1. List all active worktrees
git worktree list

# 2. Create a new worktree with a new branch (e.g. for v2.0)
git worktree add ../mds-v2 -b v2.0

# 3. Create a worktree from an existing branch
git worktree add ../mds-hotfix master

# 4. Remove a worktree folder when finished
git worktree remove ../mds-v2

# 5. Clean up stale worktree references
git worktree prune
```

---

## 4. Porting Bug Fixes between 2.0 and 1.0 (`git cherry-pick`)

### What is Cherry-Pick?
Instead of merging an entire branch (which brings all unreleased features and breaking changes), `git cherry-pick` extracts **one specific commit** and applies only its changes to your target branch.

```
[v2.0 branch]  --- (Bug Fix Commit: a1b2c3d) ---> (Continues v2.0 development...)
                                |
                        git cherry-pick a1b2c3d
                                |
                                v
[master branch] ---------------- (Applies ONLY the bug fix to v1.0)
```

---

## 5. How to Safely Cherry-Pick Without Leaking 2.0 Features

Follow this 4-step checklist to ensure 0% feature leakage:

### Step 1: Create an Atomic Commit in 2.0
Do **not** stage unrelated feature files together with the bug fix. Stage only the fix.
```powershell
# In 2.0 worktree:
git add backend/src/services/billing.service.ts
git commit -m "fix(billing): correct negative tax calculation"
```

### Step 2: Inspect the Commit Diff First
Check the exact lines changed in the commit hash:
```powershell
git show a1b2c3d
```
*Verify that only bug fix lines exist in the diff.*

### Step 3: Use `--no-commit` (`-n`) to Preview in 1.0
Apply the diff without automatically committing:
```powershell
# In c:\ai\mds (on branch master):
git cherry-pick -n a1b2c3d
```

### Step 4: Review, Test & Commit
```powershell
# Review exact changes staged in v1.0
git diff

# If clean, commit the fix:
git commit -m "fix(billing): correct negative tax calculation (cherry-picked from v2.0)"

# If unwanted changes slipped in, abort cleanly:
git cherry-pick --abort
```

### Bonus: Interactive Partial Staging (`git add -p`)
If the bug fix and feature code are mixed in the same file:
```powershell
git cherry-pick -n a1b2c3d
git reset
git add -p          # Choose 'y' for bug fix lines, 'n' for feature lines
git checkout .      # Discard remaining unstaged feature lines
git commit -m "fix: isolated bug fix"
```
