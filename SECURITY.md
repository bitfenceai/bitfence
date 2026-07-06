# Security Policy

## Reporting a Vulnerability

If you have discovered a security vulnerability in bitfence, please report it
privately. Do not open a public issue.

**Email:** security@bitfence.ai

Please include:

- A description of the vulnerability
- Steps to reproduce
- The potential impact as you understand it
- Any suggested mitigation

We will acknowledge receipt within 72 hours and provide an initial assessment
within 7 days. We treat coordinated disclosure as the default and will work
with you on a disclosure timeline appropriate to the severity.

## Scope

**In scope:**

- The bitfence scoring service and its public API endpoints
- The bitfence MCP server
- The bitfence Coinbase AgentKit action provider
- The bitfence Rig toolkit bindings

**Out of scope:**

- Vulnerabilities in upstream data providers (Helius, Alchemy, DEXScreener,
  RugCheck, honeypot.is, GoldRush) — please report those to the respective
  providers
- Third-party integrations not maintained by bitfence
- Social engineering, physical security, or attacks requiring privileged
  access to production infrastructure
