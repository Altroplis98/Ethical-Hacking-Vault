---
tags: [pentest, recon, osint, gitleaks, trufflehog, secrets, both, web]
tool: gitleaks, trufflehog
phase: 1
---
# Gitleaks and TruffleHog

Secret detection tools that scan Git repositories, commit history, and files for leaked API keys, passwords, tokens, and credentials.

[[02 - Reconnaissance and OSINT/00 - README|Folder index]]

## Gitleaks

```bash
# Install
go install github.com/gitleaks/gitleaks/v8/cmd/gitleaks@latest
# Or download from GitHub releases

# Scan a local repo
gitleaks detect -s /path/to/repo -v

# Scan repo + full commit history
gitleaks detect -s /path/to/repo --log-opts="--all" -v

# Scan a remote repo (clones first)
gitleaks detect --source="https://github.com/org/repo" -v

# Output formats
gitleaks detect -s /path/to/repo -f json -r results.json
gitleaks detect -s /path/to/repo -f csv -r results.csv

# Protect mode (pre-commit hook)
gitleaks protect -s /path/to/repo -v
```

### What gitleaks finds

- AWS keys, GCP service account keys, Azure secrets
- GitHub/GitLab tokens
- Database connection strings
- Private keys (RSA, SSH)
- Slack webhooks, Twilio tokens
- Generic passwords in config files

## TruffleHog

```bash
# Install
pip install trufflehog --break-system-packages
# Or download binary from GitHub

# Scan a GitHub repo
trufflehog github --repo=https://github.com/org/repo

# Scan a GitHub org (all repos)
trufflehog github --org=orgname --token=ghp_YOUR_TOKEN

# Scan a local filesystem
trufflehog filesystem /path/to/dir

# Scan a Git repo (local)
trufflehog git file:///path/to/repo

# Only verified secrets (actually valid)
trufflehog github --repo=https://github.com/org/repo --only-verified
```

### TruffleHog v3 advantages

- **Verification** — actually tests if discovered keys are valid
- **Multi-source** — GitHub, GitLab, S3 buckets, filesystems, Docker images
- **700+ detectors** — broader coverage than gitleaks

## Workflow

```bash
# 1. Discover GitHub repos for target org
# (use GitHub search or theHarvester)

# 2. Scan all repos
trufflehog github --org=targetorg --only-verified

# 3. For local repos (post-access)
gitleaks detect -s /opt/app/ --log-opts="--all" -f json -r /tmp/secrets.json
```

## See also

- [[20 - Wayback Machine]] — find old code/configs that may contain secrets
- [[21 - gau and katana]] — discover endpoints that might expose secrets
