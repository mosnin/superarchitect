# Security Standards — SuperArchitect OS (Global)

**Version:** 1.0
**Owner:** SuperArchitect OS Core Team
**Last Updated:** 2026-03-29
**Audience:** All agents operating within the SuperArchitect OS

---

## Purpose

This document defines the global security standards that apply to every system built by the SuperArchitect OS, regardless of system type, team, or technology stack. These standards are non-negotiable. They represent the minimum security baseline.

For detailed implementation guidance, see `os/teams/security/standards.md`.
For audit procedures, see `os/teams/security/audit-checklist.md`.

---

## Security Principles

### 1. Defense in Depth
Never rely on a single security control. Layer defenses so that failure of one control does not compromise the system. Authentication + authorization + input validation + encryption + monitoring = defense in depth.

### 2. Least Privilege
Every user, service, process, and system component gets the minimum access required to perform its function. No more. Default deny. Explicitly grant.

### 3. Secure by Default
Security controls are enabled by default, not opted into. New services are secure out of the box. Disabling security controls requires explicit justification and approval.

### 4. Zero Trust
Never trust, always verify. Every request is authenticated and authorized, regardless of network location. Internal network is not trusted. Service-to-service calls are authenticated.

### 5. Fail Secure
When a security control fails, the system denies access rather than allowing it. Authentication service down = no access. Authorization check fails = deny.

---

## Mandatory Security Controls

### Authentication
- All external endpoints require authentication
- MFA available for all human users; enforced for admin accounts
- Tokens: short-lived access tokens (30 min max), rotated refresh tokens
- Service-to-service: mTLS or signed tokens
- No custom authentication implementations without security team review

### Authorization
- Enforced at every service boundary
- Default deny: all access explicitly granted
- Resource-level authorization (prevent IDOR)
- Admin functions isolated from user interfaces

### Input Validation
- All input validated server-side
- Parameterized queries (no SQL injection)
- Output encoding (no XSS)
- Request size limits enforced
- File uploads validated (type, size, content)

### Encryption
- TLS 1.2+ for all data in transit
- AES-256 for all data at rest
- Secrets in vault (never in code, config, or environment variables)
- Key management through KMS with rotation

### Logging and Monitoring
- Authentication events logged
- Authorization failures logged
- No sensitive data in logs
- Security alerts for anomalous patterns
- Log integrity protected

### Dependency Management
- Automated vulnerability scanning in CI
- Critical CVEs block deployment
- Dependencies pinned to exact versions
- Supply chain integrity verified (lockfiles, checksums)

---

## Security Requirements by System Classification

### Internet-Facing Systems
All mandatory controls plus:
- WAF on all public endpoints
- DDoS protection
- Rate limiting on all endpoints
- CORS restricted to known origins
- CSP headers on all web responses
- Bot detection and management

### Internal Systems
All mandatory controls plus:
- Network segmentation
- Service mesh for traffic policy
- Internal CA for mTLS

### Systems Handling PII
All mandatory controls plus:
- Data classification and tagging
- Encryption at application level for restricted data
- Access logging for all PII access
- Data retention and deletion automation
- Privacy impact assessment completed

### Systems Handling Financial Data
All mandatory controls plus:
- SOC 2 controls implemented
- Audit trail for all transactions
- Non-repudiation for critical operations
- Separation of duties for financial operations

---

## Vulnerability Response SLAs

| Severity | CVSS Score | Remediation SLA |
|----------|-----------|----------------|
| Critical | 9.0-10.0 | 24 hours |
| High | 7.0-8.9 | 7 days |
| Medium | 4.0-6.9 | 30 days |
| Low | 0.1-3.9 | 90 days |

---

## Compliance Quick Reference

| Framework | Trigger | Key Requirements |
|-----------|---------|-----------------|
| SOC 2 | SaaS products, enterprise customers | Access control, monitoring, change management |
| GDPR | EU user data | Consent, DSAR, breach notification, DPA |
| HIPAA | US health data | BAA, PHI encryption, audit trails, access control |
| PCI DSS | Payment card data | Network segmentation, encryption, access control, logging |
| CCPA | California consumer data | Disclosure, opt-out, deletion rights |

---

*Security is structural. It is designed into the architecture, enforced in the code, verified in testing, and monitored in production. It is never an afterthought, never a feature flag, and never optional.*
