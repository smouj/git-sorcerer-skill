---
title: "Git Sorcerer"
description: "Enchants git workflows and resolves merge conflicts with arcane wisdom"
version: "1.0.0"
author: "FLICKCLAW-OPS"
tags: ["git", "workflow", "conflict-resolution", "version-control"]
maintainer: "SMOUJBOT"
last_updated: "2026-03-04"
type: "devops"
dependencies: ["git>=2.40", "bash>=5.0", "jq (optional)"]
environment: ["GIT_EDITOR", "GIT_MERGE_AUTOEDIT", "GIT_PAGER"]
---

# Git Sorcerer

## Purpose

Git Sorcerer automates and enchants complex git operations for the FlickClaw SaaS repository management. Real-world use cases:

- **Automated branch resurrection**: Restore deleted or orphaned branches after accidental deletion
- **Intelligent conflict arbitration**: Resolve merge conflicts using three-way diff algorithms with custom merge drivers
- **Release orchestration**: Automate version bumping, changelog generation, and tagged releases with semantic versioning
- **History purification**: Rewrite commit history to remove sensitive data while preserving ancestry
- **Submodule sorcery**: Synchronize and update nested dependencies across multiple repositories
- **Cherry-pick cascading**: Apply multiple commits across branches with automatic conflict resolution strategies
- **Hooks automation**: Deploy git hooks that enforce code review policies and run CI gates

## Scope

### Commands Managed

| Command | Primary Use | Flags/Options |
|---------|-------------|---------------|
| `git merge` | Branch integration | `--no-commit`, `--no-ff`, `--squash`, `-X ours/theirs` |
| `git rebase` | Linear history | `-i`, `--autosquash`, `--exec`, `--rebase-merges` |
| `git cherry-pick` | Commit transfer | `-x`, `--skip`, `--continue`, `--abort`, `-n` |
| `git revert` | Safe undo | `-n`, `--no-edit`, `--strategy-option` |
| `git bisect` | Bug hunting | `start`, `good`, `bad`, `view`, `skip`, `reset` |
| `git filter-branch` / `filter-repo` | History rewrite | `--tree-filter`, `--index-filter`, `--commit-filter` |
| `git worktree` | Multiple checkouts | `add`, `remove`, `list`, `prune` |
| `git submodule` | Nested repos | `update`, `sync`, `foreach`, `status` |
| `git reflog` | Recovery | `show`, `delete`, `expire` |

### Required Tools

- `git` - Core version control
- `bash` - Shell scripting
- `jq` - JSON processing (optional but recommended)
- `sed`/`awk` - Text manipulation
- `git-filter-repo` - Modern history rewriting (install via pip)

## Work Process

### 1. Initial Assessment

```bash
# Clone FlickClaw repository into isolated workspace
git clone --recurse-submodules https://github.com/smouj/flickclaw-saas.git /tmp/flickclaw-work

cd /tmp/flickclaw-work

# Analyze repository state
git status --porcelain=v1
git branch -vv
git log --oneline --graph --decorate --all -20
git remote -v
git submodule status
```

### 2. Conflict Resolution Protocol

When merge conflicts arise:

```bash
# Step 1: Identify conflict type
git diff --name-only --diff-filter=U

# Step 2: Classify conflicts
# - Text files: Use custom merge driver
# - JSON/Config: Use jq to merge objects
# - Binary: Use "theirs" or "ours" strategy

# Step 3: Execute resolution strategy
# For JSON files:
jq -s '.[0] * .[1]' file.json.base file.json.theirs > file.json

# For code files with custom markers:
git checkout --conflict=merge file.ts
# Manually edit <<<<<<< markers

# Step 4: Mark as resolved
git add file.json file.ts

# Step 5: Continue merge
git merge --continue

# Step 6: Verify resolution
git diff --check
```

### 3. Branch Resurrection Ritual

```bash
# Find lost commit (reflog)
git reflog | grep "deleted branch"

# Create branch from commit hash
git branch recovered-branch <commit-hash>

# Push if remote needed
git push origin recovered-branch
```

### 4. Release Engineering

```bash
# 1. Ensure clean state
git status
git pull origin develop

# 2. Create release branch
git checkout -b release/v1.2.3

# 3. Bump version (example for Node.js)
npm version patch -m "chore(release): %s"

# 4. Generate changelog
git log --pretty=format:'- %s' $(git describe --tags --abbrev=0)..HEAD > CHANGELOG.md

# 5. Merge to main with squash
git checkout main
git merge --squash release/v1.2.3
git commit -m "release: v1.2.3"

# 6. Tag release
git tag -a v1.2.3 -m "Release v1.2.3"

# 7. Push
git push origin main --tags
git branch -D release/v1.2.3
```

### 5. History Purification (Security)

```bash
# Install git-filter-repo if not present
pip3 install git-filter-repo

# Remove sensitive data from entire history
git filter-repo --replace-text <(echo "SECRET_KEY==>REDACTED")

# Or remove specific file
git filter-repo --path path/to/secrets.env --invert-paths

# Force push (CAUTION)
git push origin --force --all
git push origin --force --tags
```

## Golden Rules

1. **Never force push to shared branches** without explicit team approval
2. **Always create backup branch** before destructive operations:
   ```bash
   git branch backup/main-$(date +%Y%m%d-%H%M%S) main
   ```
3. **Test rebase/cherry-pick in isolated worktree** first:
   ```bash
   git worktree add /tmp/test-rebase <base-branch>
   ```
4. **Preserve merge commits** when possible (`--rebase-merges`) for audit trail
5. **Use signed commits** (`-S`) for production releases
6. **Run CI validation** before pushing any rewritten history
7. **Communicate** any force pushes to team via Slack before execution
8. **Keep** a reflog backup for 30 days minimum:
   ```bash
   git config --global gc.reflogExpireUnreachable 30.days
   ```

## Examples

### Example 1: Automated Conflict Resolution

**User Prompt:**
"Resolve merge conflicts in src/auth/ using 'theirs' strategy for binaries, manual for code"

**Execution:**
```bash
cd /tmp/flickclaw-work
git merge origin/feature/auth-refactor

# Configure merge driver for binaries (once per repo)
echo "*.png binary merge=ours" >> .gitattributes
echo "*.jpg binary merge=ours" >> .gitattributes

# For specific files:
git checkout --theirs src/auth/logo.png
git add src/auth/logo.png

# For code files, launch mergetool
git mergetool src/auth/login.ts  # Configured vimdiff or VS Code

# Complete
git merge --continue
```

**Verification:**
```bash
git diff --name-only --diff-filter=U  # Should be empty
git log -1 --pretty=fuller  # Shows merge commit with both parents
```

### Example 2: Cherry-Pick Cascade

**User Prompt:**
"Apply commits abc123, def456, ghi789 from develop to staging, skip failing ones"

**Execution:**
```bash
cd /tmp/flickclaw-work
git checkout staging

# Create script for sequential cherry-picks
cat > /tmp/cherry-script.sh << 'EOF'
#!/bin/bash
commits=("abc123" "def456" "ghi789")
for commit in "${commits[@]}"; do
  echo "Applying $commit..."
  if ! git cherry-pick $commit 2>/dev/null; then
    echo "Conflict on $commit, skipping..."
    git cherry-pick --skip
  fi
done
EOF

bash /tmp/cherry-script.sh

# Commit skipped ones manually later
```

**Verification:**
```bash
git log --oneline staging | grep -E "abc123|def456|ghi789"
git cherry -v develop staging  # Shows which commits are missing
```

### Example 3: Hotfix Production

**User Prompt:**
"Create hotfix for CVE-2024-1234 on production, merge back to develop"

**Execution:**
```bash
cd /tmp/flickclaw-work

# 1. Create hotfix branch from production tag
git checkout -b hotfix/CVE-2024-1234 production/v2.1.0

# 2. Apply security patch
# (edit files...)

git add .
git commit -S -m "fix(security): patch CVE-2024-1234

- Close potential RCE vulnerability in user upload
- Sanitize filename extensions
- Add content-type validation"

# 3. Bump version
npm version patch -m "fix(security): %s [CVE-2024-1234]"

# 4. Merge to production
git checkout production
git merge --no-ff hotfix/CVE-2024-1234 -m "release: hotfix CVE-2024-1234"
git tag -s v2.1.1 -m "Security hotfix v2.1.1"

# 5. Merge back to develop
git checkout develop
git merge --no-ff hotfix/CVE-2024-1234

# 6. Push all
git push origin production --tags
git push origin develop
git push origin hotfix/CVE-2024-1234

# 7. Cleanup
git branch -D hotfix/CVE-2024-1234
```

### Example 4: Recover Lost Work

**User Prompt:**
"Recover commits from yesterday that were lost after a bad reset"

**Execution:**
```bash
cd /tmp/flickclaw-work

# Search reflog for dangling commits
git reflog --all --since="1 day ago" | head -20

# Find commit hash (e.g., abc1234)
git show abc1234  # Verify it's the right one

# Create branch
git branch recovered/feature-work abc1234

# If commit not in reflog, use fsck
git fsck --lost-found | grep commit
cat .git/lost-found/commit/* | git show  # Browse recovered

# Push to remote for safety
git push origin recovered/feature-work
```

## Rollback Commands

### Merge Rollback
```bash
# Abort merge in progress
git merge --abort

# If merge already committed
git revert -m 1 <merge-commit-hash>  # Keep both parents
# OR for complete undo:
git reset --hard ORIG_HEAD  # CAUTION: discards working tree
```

### Rebase Rollback
```bash
# Abort rebase
git rebase --abort

# If rebase completed and pushed
git revert <oldest-rebased-commit>..HEAD
# OR (dangerous):
git reset --hard <commit-before-rebase>
git push --force-with-lease  # If remote updated
```

### Cherry-Pick Rollback
```bash
git cherry-pick --abort  # If in progress
git revert <cherry-picked-commit>  # If completed
```

### Filter-Repo/Rewrite Rollback
```bash
# If you backed up first:
git checkout backup/pre-filter-<timestamp>

# If not, recover from reflog within 30 days:
git reflog | grep "filter-repo"
git checkout -b recovery <reflog-hash>
```

### Submodule Rollback
```bash
git submodule deinit -f path/to/submodule
git rm -f path/to/submodule
rm -rf .git/modules/path/to/submodule
git commit -m "chore: remove broken submodule"
```

### Complete Project Reset (Nuclear)
```bash
# WARNING: Irreversible without remote backup
git reset --hard $(git reflog | head -1 | cut -d' ' -f1)
git clean -fdx  # Remove untracked files too
git submodule foreach --recursive git reset --hard
git submodule foreach --recursive git clean -fdx
```

## Troubleshooting

### "Refusing to merge unrelated histories"
```bash
# Use with caution - only if repositories were independently created
git merge other-branch --allow-unrelated-histories
```

### "error: Your local changes to the following files would be overwritten by merge"
```bash
# Option A: Stash changes
git stash push -m "pre-merge-stash"
git merge <branch>

# Option B: Force checkout (discard changes)
git checkout --force <branch>

# Option C: Merge with ours strategy for specific file
git checkout --ours path/to/file
```

### "fatal: refusing to merge unrelated histories" with submodules
```bash
# Initialize submodules properly
git submodule update --init --recursive
git submodule sync --recursive
```

### "detached HEAD" state after cherry-pick
```bash
# Create branch to preserve work
git checkout -b temp/cherry-pick-work
# Then merge or rebase as needed
```

### "Permission denied (publickey)" when pushing
```bash
# Verify SSH agent
ssh-add -l  # List loaded keys
ssh-add ~/.ssh/id_rsa_flickclaw  # Add correct key

# Check remote URL
git remote -v
# If using HTTPS but want SSH:
git remote set-url origin git@github.com:smouj/flickclaw-saas.git
```

### Merge conflicts in `.gitmodules`
```bash
# Treat as ini file
git config --global merge.renormalize true
git checkout --ours .gitmodules
git add .gitmodules
```

## Verification Steps

After any git operation:

1. **Check branch state**:
   ```bash
   git branch -vv  # All branches with upstream tracking
   git status -sb   # Short status with branch info
   ```

2. **Validate commit graph**:
   ```bash
   git log --graph --oneline --all -30
   # Look for unexpected divergences or lost commits
   ```

3. **Verify no uncommitted changes**:
   ```bash
   git diff-index --quiet HEAD --  # Returns 0 if clean
   git status --porcelain | wc -l  # Should be 0
   ```

4. **Test build/test suite**:
   ```bash
   npm test
   npm run build
   # Or project-specific: pnpm test, yarn build, etc.
   ```

5. **Validate remote sync**:
   ```bash
   git fetch origin
   git log --oneline HEAD..origin/main  # Should be empty if up-to-date
   ```

6. **Check reflog integrity**:
   ```bash
   git reflog --date=iso | head -10
   ```

## Environment Variables

| Variable | Purpose | Default | Required |
|----------|---------|---------|----------|
| `GIT_EDITOR` | Commit message editor | `vim` | No |
| `GIT_MERGE_AUTOEDIT` | Auto-open editor on merge | `yes` | No |
| `GIT_PAGER` | Pager for logs/diffs | `less` | No |
| `GIT_SSH_COMMAND` | Custom SSH for git | - | No |
| `GIT_USER_NAME` | Override git user.name | from config | No |
| `GIT_USER_EMAIL` | Override git user.email | from config | No |

Setup example:
```bash
export GIT_EDITOR="code --wait"
export GIT_MERGE_AUTOEDIT=no  # For automated workflows
export GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa_flickclaw -o IdentitiesOnly=yes"
```

## Dependencies & Setup

### Prerequisites
```bash
# Verify git version
git --version  # Must be >= 2.40

# Install git-filter-repo (for history rewriting)
pip3 install --user git-filter-repo

# Optional: jq for JSON manipulation
apt-get install jq  # Debian/Ubuntu
brew install jq     # macOS

# Configure git for this project
git config --local user.name "FlickClaw Bot"
git config --local user.email "devops@flickclaw.com"
git config --local commit.gpgsign true
```

### One-time Setup for FlickClaw Repository
```bash
cd /tmp/flickclaw-work

# Set upstream tracking
git branch --set-upstream-to=origin/main main

# Configure merge drivers for common file types
cat > .gitattributes << 'EOF'
*.json merge=json
*.md merge=union
*.lock merge=ours
*.png binary
*.jpg binary
EOF

# Install custom merge driver for JSON
git config --local merge.json.name "JSON merge driver"
git config --local merge.json.driver "jq -s '.[0] * .[1]' %A %O > %A"
```

## Notes

- All operations should run in `/tmp/flickclaw-work` to avoid corruption of working OpenClaw environment
- Always create backup branches before destructive operations
- For production deployments, use `--force-with-lease` instead of `--force` when pushing rewritten history
- Coordinate with team before rewriting shared branch history
- Keep this skill idempotent - running the same operation twice should not cause side effects
- Log all operations to `/tmp/flickclaw-work/git-sorcerer.log` for audit trail
```