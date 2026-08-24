# GitHub Merge & Combine — CLI Guide

Combine local changes and PRs using the GitHub CLI (`gh`) and Git.

---

## 1. Three Common Scenarios

The right command depends on what state your changes are in.

### Scenario A — Squash local commits before pushing
Use when you have multiple local commits you want to merge into one before pushing (e.g. several "WIP" commits → one clean commit on `main`).

### Scenario B — Squash a feature branch into `main` via PR
Use when you want to combine all commits on a feature branch into a single commit when merging.

### Scenario C — Update an existing PR with a squash
Use when a PR is already open and you want to replace its commits with one squashed commit.

---

## 2. Command Reference

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 6%; text-align: center;">#</th>
      <th style="width: 24%;">Action</th>
      <th style="width: 35%;">GitHub CLI Command</th>
      <th style="width: 35%;">Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center;">1</td>
      <td>Check repo / branch</td>
      <td><code>gh repo view</code></td>
      <td>Confirms you're in the right repo</td>
    </tr>
    <tr>
      <td style="text-align: center;">2</td>
      <td>Check status</td>
      <td><code>gh status</code></td>
      <td>Shows open PRs/issues assigned to you</td>
    </tr>
    <tr>
      <td style="text-align: center;">3</td>
      <td>List branches</td>
      <td><code>gh branch list</code></td>
      <td>All local + remote branches</td>
    </tr>
    <tr>
      <td style="text-align: center;">4</td>
      <td>List PRs</td>
      <td><code>gh pr list</code></td>
      <td>Open PRs in current repo</td>
    </tr>
    <tr>
      <td style="text-align: center;">5</td>
      <td>View a PR</td>
      <td><code>gh pr view 123</code></td>
      <td>Inspect PR by number</td>
    </tr>
    <tr>
      <td style="text-align: center;">6</td>
      <td>Create PR (default merge)</td>
      <td><code>gh pr create --fill</code></td>
      <td>Uses default merge strategy</td>
    </tr>
    <tr>
      <td style="text-align: center;">7</td>
      <td>Create PR — squash merge</td>
      <td><code>gh pr create --fill</code></td>
      <td>Enable squash option during merge</td>
    </tr>
    <tr>
      <td style="text-align: center;">8</td>
      <td>Merge PR with squash</td>
      <td><code>gh pr merge 123 --squash --delete-branch</code></td>
      <td>Squash + delete head branch in one shot</td>
    </tr>
    <tr>
      <td style="text-align: center;">9</td>
      <td>Merge PR (rebase)</td>
      <td><code>gh pr merge 123 --rebase --delete-branch</code></td>
      <td>Linear history, no merge commit</td>
    </tr>
    <tr>
      <td style="text-align: center;">10</td>
      <td>Merge PR (merge commit)</td>
      <td><code>gh pr merge 123 --merge --delete-branch</code></td>
      <td>Preserves all commits + merge commit</td>
    </tr>
    <tr>
      <td style="text-align: center;">11</td>
      <td>Squash last N local commits</td>
      <td><code>git rebase -i HEAD~N</code></td>
      <td>Change <code>pick</code> to <code>squash</code> (pure Git)</td>
    </tr>
    <tr>
      <td style="text-align: center;">12</td>
      <td>Push force after squash</td>
      <td><code>git push --force-with-lease</code></td>
      <td>Safer than raw <code>--force</code></td>
    </tr>
  </tbody>
</table>

> [!NOTE]
> `gh pr merge` flags `--squash`, `--rebase`, and `--merge` are **mutually exclusive** — select one per merge.

---

## 3. Step-by-Step Walkthroughs

### A. Squash local commits into one before pushing

```bash
# 1. See what you have
git log --oneline -5

# 2. Interactive rebase the last N commits
#    In the editor, change "pick" to "squash" (or "s") on commits you want to fold
git rebase -i HEAD~3

# 3. Rewrite the combined commit message, save, exit

# 4. Verify
git log --oneline -3

# 5. Push safely (refuses if remote has new commits you didn't pull)
git push --force-with-lease
```

### B. Squash-merge a feature branch into `main`

```bash
# 1. Make sure main is up to date and you're on the feature branch
gh repo sync                    # optional: sync fork if applicable
git checkout main
git pull
git checkout feature/my-change

# 2. Push the feature branch (if not already)
git push -u origin feature/my-change

# 3. Create the PR
gh pr create --fill --base main

# 4. After review/approval, squash-merge it (collapses all feature commits into 1 on main)
gh pr merge --squash --delete-branch

# 5. Update local main
git checkout main
git pull
```

### C. Squash an existing open PR

```bash
# 1. List open PRs to find the number
gh pr list

# 2. Squash-merge it directly (no need to update the branch first if you don't care about history)
gh pr merge 42 --squash --delete-branch

# Optional: confirm
gh pr view 42 --json state,mergedAt,mergeCommit
```

---

## 4. Repo-Specific Example

For a repo at `C:\ai\open-antigravity-patcher`:

### Squash local commits into one before pushing

```bash
cd "C:/ai/open-antigravity-patcher"

# 1. See recent commits
git log --oneline -10

# 2. If the last 3 should become 1
git rebase -i HEAD~3
#    In editor: leave the oldest as "pick", change the others to "squash"

# 3. Force-push (safe variant)
git push --force-with-lease origin main
```

### Combine a feature branch into `main` via squash-merge PR

```bash
cd "C:/ai/open-antigravity-patcher"

# 1. Create & switch to a feature branch
git checkout -b feature/readme-english

# 2. Make your change, commit
git add README.md
git commit -m "Translate README to English"

# 3. Push and open PR
git push -u origin feature/readme-english
gh pr create --fill --base main

# 4. Squash-merge via CLI
gh pr merge --squash --delete-branch
```

---

## 5. Safety Reminders

- **`git push --force-with-lease`** is always safer than `--force` — it refuses to push if the remote has commits you haven't seen.
- **Squash-merging a PR** rewrites history on the base branch but leaves the source branch alone until you delete it.
- **Rebasing a branch you share with others** rewrites their view of history too — coordinate before rebasing shared branches.
- **`gh pr merge --delete-branch`** only deletes the **remote** branch. Run `git branch -d feature/my-change` locally to clean up.
