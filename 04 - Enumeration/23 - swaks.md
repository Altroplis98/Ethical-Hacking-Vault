---
tags: [pentest, enumeration, smtp, swaks, email, recon]
tool: swaks
phase: 3
---
# swaks

Swiss Army Knife for SMTP. Test mail servers, send test emails, verify relay configs, test auth.

[[04 - Enumeration/00 - README|Folder index]]

## Install / verify

```bash
which swaks || sudo apt install swaks -y
```

## Usage

```bash
# Test basic connectivity
swaks --to user@example.com --server 10.10.10.10

# Test open relay
swaks --from attacker@evil.com --to victim@example.com --server 10.10.10.10

# Authenticated send
swaks --to user@example.com --server 10.10.10.10 --auth LOGIN --auth-user user --auth-password pass

# Custom body
swaks --to user@example.com --server 10.10.10.10 --body "Test message"

# Attach file
swaks --to user@example.com --server 10.10.10.10 --attach /path/to/file.pdf

# TLS
swaks --to user@example.com --server 10.10.10.10 --tls
```

## Testing open relay

An open relay allows you to send email as anyone to anyone — useful for phishing in pentests.

```bash
swaks --from ceo@target.com --to employee@target.com --server 10.10.10.10 --header "Subject: Urgent" --body "Click here"
# If it succeeds without auth, the server is an open relay
```

## See also

- [[22 - smtp-user-enum]] — enumerate valid users before testing
