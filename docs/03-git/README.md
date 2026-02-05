# Version Control (Git)

[← Back to Index](../../README.md)

This section covers Git fundamentals including branching, merging, rebasing, conflict resolution, and workflows.

---

<details>
<summary><strong>Q1: What is Git and how does it differ from other VCS?</strong></summary>

Git is a **distributed version control system** (DVCS) that tracks changes in source code during software development.

| Feature | Git (Distributed) | SVN (Centralized) |
|---------|-------------------|-------------------|
| **Repository** | Full copy on every machine | Single central server |
| **Offline work** | Full functionality offline | Requires server connection |
| **Speed** | Fast (local operations) | Slower (network dependent) |
| **Branching** | Lightweight, fast | Heavy, slow |
| **Backup** | Every clone is a backup | Single point of failure |

**Key Concepts:**
- **Working Directory** - Where you edit files
- **Staging Area (Index)** - Files ready to commit
- **Repository (.git)** - Commit history and metadata

</details>

<details>
<summary><strong>Q2: Explain Git branching and common branch strategies</strong></summary>

A branch is a lightweight pointer to a commit, allowing parallel development.

```bash
# Create and switch to branch
git checkout -b feature/login

# List branches
git branch -a

# Delete branch
git branch -d feature/login
git branch -D feature/login  # Force delete
```

**Common Branching Strategies:**

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Git Flow** | main, develop, feature, release, hotfix | Scheduled releases |
| **GitHub Flow** | main + feature branches, deploy from main | Continuous deployment |
| **Trunk-Based** | Short-lived branches, merge frequently | CI/CD, small teams |

</details>

<details>
<summary><strong>Q3: What is the difference between merge and rebase?</strong></summary>

Both integrate changes from one branch into another, but differently:

| Aspect | Merge | Rebase |
|--------|-------|--------|
| **History** | Preserves all commits, creates merge commit | Rewrites history, linear |
| **Conflicts** | Resolve once | Resolve per commit |
| **Use case** | Public branches, preserving context | Clean up local commits |

```bash
# Merge: Creates merge commit
git checkout main
git merge feature

# Rebase: Replays commits on top of main
git checkout feature
git rebase main
```

**Golden Rule:** Never rebase public/shared branches!

**Interactive Rebase (cleanup commits):**
```bash
git rebase -i HEAD~3  # Edit last 3 commits
# Options: pick, squash, reword, drop, fixup
```

</details>

<details>
<summary><strong>Q4: What is cherry-pick and when would you use it?</strong></summary>

Cherry-pick applies a **specific commit** from one branch to another without merging the entire branch.

```bash
# Apply single commit
git cherry-pick abc1234

# Apply multiple commits
git cherry-pick abc1234 def5678

# Cherry-pick without committing
git cherry-pick abc1234 --no-commit
```

**Use Cases:**
- Hotfix a bug in production without merging unfinished features
- Port a specific feature to a different branch
- Undo an accidental commit to the wrong branch

</details>

<details>
<summary><strong>Q5: Explain git reset vs git revert</strong></summary>

| Command | What it does | History | Safe for shared branches? |
|---------|--------------|---------|---------------------------|
| `git reset` | Moves HEAD, can discard commits | Rewrites history | ❌ No |
| `git revert` | Creates new commit undoing changes | Preserves history | ✅ Yes |

**Reset Modes:**
```bash
git reset --soft HEAD~1   # Keep changes staged
git reset --mixed HEAD~1  # Keep changes unstaged (default)
git reset --hard HEAD~1   # Discard all changes ⚠️
```

**Revert (safe for shared branches):**
```bash
git revert abc1234        # Create commit that undoes abc1234
git revert HEAD~3..HEAD   # Revert last 3 commits
```

</details>

<details>
<summary><strong>Q6: What is git stash and how do you use it?</strong></summary>

Stash temporarily saves uncommitted changes so you can switch branches.

```bash
# Save changes to stash
git stash
git stash push -m "work in progress"

# List stashes
git stash list

# Apply and keep in stash
git stash apply stash@{0}

# Apply and remove from stash
git stash pop

# Drop specific stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch stash@{0}
```

**Use Cases:**
- Switch branches with uncommitted work
- Pull changes when you have local modifications
- Temporarily set aside experimental changes

</details>

<details>
<summary><strong>Q7: Explain .gitignore and common patterns</strong></summary>

`.gitignore` specifies files/directories Git should ignore.

```gitignore
# Dependencies
node_modules/
vendor/

# Build outputs
dist/
build/
*.exe

# Environment files
.env
.env.local

# IDE/Editor
.idea/
.vscode/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Patterns
*.tmp           # All .tmp files
!important.tmp  # Except this one
**/cache/       # cache/ in any directory
```

**Already tracked files:**
```bash
# Stop tracking file but keep locally
git rm --cached filename

# Stop tracking entire directory
git rm -r --cached directory/
```

</details>

<details>
<summary><strong>Q8: What is Git Flow workflow?</strong></summary>

Git Flow is a branching model with defined branch types:

| Branch | Purpose | Merges Into |
|--------|---------|-------------|
| **main** | Production-ready code | - |
| **develop** | Integration branch | main (via release) |
| **feature/** | New features | develop |
| **release/** | Release preparation | main & develop |
| **hotfix/** | Production fixes | main & develop |

```bash
# Start feature
git checkout develop
git checkout -b feature/user-auth

# Finish feature
git checkout develop
git merge --no-ff feature/user-auth

# Start release
git checkout -b release/1.0 develop

# Finish release
git checkout main
git merge --no-ff release/1.0
git tag -a v1.0

# Hotfix
git checkout -b hotfix/critical-bug main
# ... fix bug ...
git checkout main && git merge hotfix/critical-bug
git checkout develop && git merge hotfix/critical-bug
```

</details>

<details>
<summary><strong>Q9: How do you resolve merge conflicts?</strong></summary>

Conflicts occur when Git can't auto-merge changes.

**Conflict markers in file:**
```
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> feature-branch
```

**Resolution steps:**
```bash
# 1. Identify conflicted files
git status

# 2. Open and edit files, remove markers

# 3. Stage resolved files
git add resolved-file.txt

# 4. Complete merge
git commit -m "Resolve merge conflicts"

# Or abort merge
git merge --abort
```

**Tools:**
```bash
# Use merge tool
git mergetool

# VS Code, IntelliJ have built-in conflict resolution
```

</details>

<details>
<summary><strong>Q10: What is git reflog and when is it useful?</strong></summary>

Reflog records when branch tips and HEAD are updated. It's your **safety net** for recovering lost commits.

```bash
# View reflog
git reflog

# Output example:
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: checkout: moving from main to feature
# 789abcd HEAD@{2}: reset: moving to HEAD~1

# Recover "lost" commit after hard reset
git reset --hard HEAD@{2}

# Recover deleted branch
git checkout -b recovered-branch abc1234
```

**Use Cases:**
- Recover from accidental `git reset --hard`
- Find commits from deleted branches
- Undo a bad rebase

**Note:** Reflog is local only and expires (default 90 days).

</details>

<details>
<summary><strong>Q11: Explain Git hooks</strong></summary>

Git hooks are scripts that run automatically at certain points in Git workflow.

| Hook | When | Use Case |
|------|------|----------|
| `pre-commit` | Before commit | Lint, format code, run tests |
| `commit-msg` | After commit message entered | Validate commit message format |
| `pre-push` | Before push | Run tests, check branch policy |
| `post-merge` | After merge | Install dependencies |
| `pre-rebase` | Before rebase | Prevent rebasing shared branches |

**Location:** `.git/hooks/`

**Example pre-commit hook:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Commit aborted."
  exit 1
fi
```

**Team sharing:** Use tools like **Husky** (JS) or **pre-commit** (Python) to version-control hooks.

</details>

<details>
<summary><strong>Q12: Common Git troubleshooting commands</strong></summary>

```bash
# Undo last commit, keep changes
git reset --soft HEAD~1

# Amend last commit message
git commit --amend -m "New message"

# Add forgotten file to last commit
git add forgotten-file
git commit --amend --no-edit

# Undo uncommitted changes in file
git checkout -- filename
git restore filename  # Modern

# Discard all local changes
git checkout -- .
git clean -fd  # Remove untracked files/dirs

# Find which commit introduced a bug
git bisect start
git bisect bad HEAD
git bisect good v1.0

# See who changed each line
git blame filename

# Find commits by message
git log --grep="bug fix"

# Find commits changing specific code
git log -S "functionName" --source --all
```

</details>

