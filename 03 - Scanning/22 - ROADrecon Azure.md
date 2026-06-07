---
tags: [pentest, scanning, cloud, azure, roadrecon, ad, both, recon]
tool: roadrecon
phase: 2
---
# ROADrecon Azure

Azure AD enumeration and exploration tool. Dumps and visualizes the entire Azure AD tenant — users, groups, apps, roles, permissions.

[[03 - Scanning/00 - README|Folder index]]

## Install

```bash
pip install roadrecon --break-system-packages
```

## Usage

```bash
# Authenticate (various methods)
roadrecon auth -u user@tenant.onmicrosoft.com -p 'password'
roadrecon auth --access-token <token>
roadrecon auth --device-code  # device code flow

# Gather all Azure AD data
roadrecon gather

# Start the web UI to explore
roadrecon gui
# Browse to http://127.0.0.1:5000
```

## What it enumerates

| Data | Value |
| --- | --- |
| Users | Full user list with attributes, MFA status |
| Groups | Group memberships, nested groups |
| Applications | Registered apps + permissions (API scopes) |
| Service Principals | App identities + assigned roles |
| Roles | Directory roles and who holds them |
| Devices | Registered/joined devices |
| OAuth2 permissions | What apps can access |

## ROADrecon GUI features

- Visual graph of user-group-role relationships
- Application permission analysis
- Device-to-user mapping
- Export to BloodHound-compatible format (with ROADtools)

## When to use

- Post-authentication Azure AD enumeration
- Mapping attack paths through Azure AD roles and permissions
- Identifying over-permissioned applications
- Finding service accounts with interesting permissions

## See also

- [[20 - ScoutSuite]] — broader cloud config audit
- [[21 - Prowler AWS]] — AWS-specific alternative
