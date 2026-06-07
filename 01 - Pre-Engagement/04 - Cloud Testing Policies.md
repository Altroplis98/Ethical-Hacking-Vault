---
tags: [pentest, pre-engagement, cloud, aws, azure, gcp]
phase: 0
---

# Cloud Testing Policies

Each major cloud provider has its own rules for penetration testing. Violating them can get the client's account suspended.

[[01 - Pre-Engagement/00 - README|Folder index]]

## AWS

As of 2024, AWS **does not require pre-approval** for pen testing against most services you own. Permitted services include EC2, RDS, Aurora, CloudFront, API Gateway, Lambda, Lightsail, Elastic Beanstalk, and more.

### Prohibited on AWS

- DNS zone walking via Route 53 hosted zones
- DoS, DDoS, or simulated DoS/DDoS
- Port flooding
- Protocol flooding
- Request flooding (login, API, etc.)

### AWS pen test request form

If your testing might resemble prohibited activity or you want written confirmation:
`https://aws.amazon.com/security/penetration-testing/`

## Azure / Microsoft 365

Microsoft **does not require pre-notification** for pen testing against your own Azure resources as of 2024. Their rules:

### Permitted

- Testing against resources you own in your Azure subscription
- Web app testing against your own App Services
- Network testing within your own VNets

### Prohibited

- Testing other customers' resources
- DoS testing of any kind
- Port scanning that generates excessive traffic
- Any testing that could impact shared infrastructure

### M365 / Entra ID

- Phishing simulations: use Microsoft Attack Simulation Training (built-in)
- Testing against Entra ID authentication endpoints: permitted if you own the tenant
- Do NOT brute-force login.microsoftonline.com at scale — rate limits will trigger and Microsoft may flag the tenant

## GCP

Google **requires you to follow their Acceptable Use Policy** but does not require pre-notification for testing your own projects.

### Key restrictions

- No DoS testing
- No testing that impacts other GCP customers
- Vulnerability scanning of Compute Engine, GKE, App Engine — permitted on your own resources
- If in doubt, review: `https://cloud.google.com/terms/aup`

## General cloud testing checklist

```text
[ ] Confirm client owns the cloud account / subscription / project
[ ] Review current provider pen test policy (policies change!)
[ ] Get written client authorization naming specific cloud resources
[ ] Confirm no shared/multi-tenant resources are in scope
[ ] Set up your attack box OUTSIDE the target cloud account
[ ] Document all cloud-specific tools used (ScoutSuite, Prowler, Pacu)
[ ] Avoid triggering GuardDuty / Defender / SCC alerts unnecessarily
    (or coordinate with client's SOC so they expect it)
```

> [!tip] Use the provider's own security tools first
> AWS Inspector, Azure Defender, GCP Security Command Center — running these first gives you a baseline and the client often already has them enabled.

## See also

- [[03 - Scope Definition]] — general scoping that includes cloud
- [[05 - Attack Infrastructure Setup]] — setting up your attack box
