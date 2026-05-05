# 🔀 Git for Cloud Security — Essential Commands

> A practical reference of Git commands used in Cloud Security workflows —
> covering secure development, auditing, secrets management, and team collaboration.

---

## 📋 Table of Contents

- [Initial Setup and Hardening](#initial-setup-and-hardening)
- [Repository Management](#repository-management)
- [Branching and Secure Workflow](#branching-and-secure-workflow)
- [Staging and Committing Safely](#staging-and-committing-safely)
- [Commit Signing with GPG](#commit-signing-with-gpg)
- [Auditing and History Investigation](#auditing-and-history-investigation)
- [Secrets and Sensitive Data](#secrets-and-sensitive-data)
- [Remote and Access Control](#remote-and-access-control)
- [CICD Integration](#cicd-integration)
- [Incident Response](#incident-response)
- [Quick Reference](#quick-reference)
- [Recommended Tools](#recommended-tools)

---

## Initial Setup and Hardening

```bash
# Identity configuration (always set before committing)
git config --global user.name "Your Name"
git config --global user.email "you@company.com"

# Force GPG signing on all commits globally
git config --global commit.gpgsign true

# Set default branch name to main
git config --global init.defaultBranch main

# Enable credential helper to avoid storing passwords in plaintext
git config --global credential.helper store

# View all global configurations
git config --global --list

# Set editor
git config --global core.editor "nano"
```

---

## Repository Management

```bash
# Initialize a new repo
git init

# Clone a repo — prefer SSH in production
git clone git@github.com:org/repo.git
git clone https://github.com/org/repo.git

# Check repo status
git status

# View remote URLs
git remote -v

# Add a remote
git remote add origin git@github.com:org/repo.git

# Remove a remote
git remote remove origin
```

---

## Branching and Secure Workflow

Working on isolated branches prevents accidental exposure of sensitive changes to `main`.

```bash
# List all branches (local and remote)
git branch -a

# Create and switch to a new branch
git checkout -b feature/security-patch

# Switch between branches
git checkout main

# Delete a local branch after merge
git branch -d feature/security-patch

# Force delete a branch (use with caution)
git branch -D feature/security-patch

# Merge a branch into main
git checkout main
git merge feature/security-patch

# Rebase (cleaner history)
git rebase main
```

---

## Staging and Committing Safely

```bash
# Stage a specific file
git add path/to/file.tf

# Stage all changes (review first with git status)
git add .

# Review what is staged before committing
git diff --staged

# Commit with a descriptive message
git commit -m "fix: remove hardcoded AWS credentials from config"

# Amend the last commit message (before pushing)
git commit --amend -m "fix: revoke and rotate exposed credentials"

# Unstage a file without losing changes
git restore --staged path/to/file

# Discard local changes in a file
git restore path/to/file
```

---

## Commit Signing with GPG

Signed commits prove that code was authored by a verified identity — critical in regulated environments.

```bash
# List available GPG keys
gpg --list-secret-keys --keyid-format=long

# Configure Git to use your GPG key
git config --global user.signingkey YOUR_KEY_ID

# Sign a single commit manually
git commit -S -m "feat: add IAM role hardening policy"

# Enable automatic signing for all commits
git config --global commit.gpgsign true

# Verify the signature of a commit
git log --show-signature -1

# Verify signatures across recent commits
git log --show-signature --oneline -10
```

---

## Auditing and History Investigation

Essential for forensic analysis, compliance reviews, and incident investigations.

```bash
# View full commit history
git log

# Compact one-line log with graph
git log --oneline --graph --decorate --all

# Search commit history by keyword
git log --all --grep="password"
git log --all --grep="secret"
git log --all --grep="key"

# Show who changed what line
git blame path/to/file.py

# Show what changed in a specific commit
git show <commit-hash>

# Compare two commits or branches
git diff <commit-A> <commit-B>

# Find when a bug was introduced (binary search)
git bisect start
git bisect bad
git bisect good <commit-hash>
git bisect reset

# View all activity in the repo including resets and rebases
git reflog
```

---

## Secrets and Sensitive Data

If credentials or secrets are accidentally committed, act immediately.

```bash
# Check if a file is tracked by Git
git ls-files path/to/.env

# Remove a file from tracking WITHOUT deleting it locally
git rm --cached path/to/.env

# Add secrets files to .gitignore
echo ".env" >> .gitignore
echo "*.pem" >> .gitignore
echo "*.key" >> .gitignore
echo "terraform.tfvars" >> .gitignore
git add .gitignore
git commit -m "chore: add secrets files to .gitignore"

# Purge a file from ALL history — modern approach (recommended)
pip install git-filter-repo
git filter-repo --path path/to/secrets.txt --invert-paths

# After purging history, force push to remote
git push origin --force --all
git push origin --force --tags
```

> **Warning:** Purging history does not revoke credentials. Always rotate and invalidate any exposed secrets immediately.

---

## Remote and Access Control

```bash
# Fetch updates without merging
git fetch origin

# Pull latest changes from main
git pull origin main

# Push changes to remote
git push origin feature/security-patch

# Push and set upstream tracking
git push -u origin feature/security-patch

# Force push (use carefully — only on your own branches)
git push --force origin feature/security-patch

# List all tags
git tag

# Create a signed tag for a release
git tag -s v1.0.0 -m "Release v1.0.0"

# Push tags to remote
git push origin --tags

# Verify a signed tag
git tag -v v1.0.0
```

---

## CICD Integration

Git integrates with tools like GitHub Actions, GitLab CI, and Terraform Cloud.

```bash
# Create a deploy key (read-only SSH key for CI/CD runners)
ssh-keygen -t ed25519 -C "ci-deploy-key" -f ~/.ssh/deploy_key

# Clone using SSH in a CI environment
GIT_SSH_COMMAND="ssh -i ~/.ssh/deploy_key" git clone git@github.com:org/repo.git

# Shallow clone for faster CI pipelines
git clone --depth 1 git@github.com:org/repo.git

# Create an archive of the repo for artifact deployment
git archive --format=tar.gz --output=release.tar.gz HEAD

# Check for merge conflicts before merging in scripts
git merge --no-commit --no-ff feature/branch
git merge --abort
```

---

## Incident Response

When a security incident involves a Git repository.

```bash
# Preserve current repo state before any changes
git stash
git log --all --oneline > incident_log.txt

# Find all commits by a specific author
git log --all --author="suspected@email.com"

# Search all commits for a specific string across history
git log -S "ACCESS_KEY" --all --source --oneline

# Revert a dangerous commit without rewriting history
git revert <commit-hash>

# Create a snapshot tag before incident remediation
git tag incident/2025-04-28 HEAD

# Reset branch to a safe state (destructive)
git reset --hard <last-safe-commit-hash>
git push --force origin main
```

---

## Quick Reference

| Practice | Command |
|---|---|
| Never commit secrets | `.gitignore` + `git rm --cached` |
| Sign all commits | `git config --global commit.gpgsign true` |
| Prefer SSH over HTTPS | `git clone git@github.com:...` |
| Audit commit history | `git log --all --grep="secret"` |
| Verify author identity | `git log --show-signature` |
| Rotate before purging | Revoke credentials first, then `git filter-repo` |
| Use branch protection | Configure in GitHub or GitLab settings |
| Shallow clones for CI | `git clone --depth 1` |

---

## Recommended Tools

| Tool | Purpose |
|---|---|
| [truffleHog](https://github.com/trufflesecurity/trufflehog) | Scan Git history for leaked secrets |
| [gitleaks](https://github.com/gitleaks/gitleaks) | Detect hardcoded credentials in repos |
| [git-secrets](https://github.com/awslabs/git-secrets) | Prevent committing secrets (AWS) |
| [pre-commit](https://pre-commit.com/) | Run security hooks before committing |
| [Checkov](https://www.checkov.io/) | Static analysis for IaC (Terraform, etc.) |

---

<div align="center">

*Maintained as a personal reference for Cloud Security practices.*
*Commands tested on Linux environment.*

</div>
