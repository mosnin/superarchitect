# Security Audit Checklist — SuperArchitect OS

**Version:** 1.0
**Owner:** Security Agent (Team 6)
**Last Updated:** 2026-03-29

---

## Purpose

This checklist is used by the Security Agent to conduct comprehensive security audits. Every item must be evaluated as PASS, FAIL, N/A, or PARTIAL with notes. A system with any FAIL on a Critical item is not production-ready.

---

## 1. Architecture Security Review

### 1.1 Authentication Architecture
- [ ] **[Critical]** Centralized identity provider in use (not custom auth)
- [ ] **[Critical]** All external endpoints require authentication
- [ ] **[Critical]** Token validation checks signature, expiry, issuer, and audience
- [ ] **[High]** Access tokens are short-lived (< 30 minutes)
- [ ] **[High]** Refresh tokens use rotation (single-use)
- [ ] **[High]** MFA available for all human users
- [ ] **[Critical]** MFA enforced for administrative accounts
- [ ] **[High]** Service-to-service authentication via mTLS or signed tokens
- [ ] **[Medium]** Token storage follows platform best practices (HTTP-only cookies for web)
- [ ] **[High]** Authentication failure handling does not reveal account existence

### 1.2 Authorization Architecture
- [ ] **[Critical]** Authorization enforced at every service boundary
- [ ] **[Critical]** Default deny: access denied unless explicitly granted
- [ ] **[Critical]** Resource-level authorization prevents IDOR attacks
- [ ] **[High]** Centralized policy engine (OPA, Cedar, or equivalent)
- [ ] **[High]** Principle of least privilege applied to all service accounts
- [ ] **[Medium]** Permission model documented and reviewable
- [ ] **[High]** Administrative functions isolated from user-facing interfaces
- [ ] **[Medium]** Temporal access controls for elevated privileges

### 1.3 Data Flow Security
- [ ] **[Critical]** All data in transit encrypted (TLS 1.2+)
- [ ] **[Critical]** All data at rest encrypted (AES-256 or equivalent)
- [ ] **[High]** PII identified and classified in data model
- [ ] **[High]** Sensitive data does not appear in logs
- [ ] **[High]** Sensitive data does not appear in URLs
- [ ] **[Medium]** Data retention policies defined and automated
- [ ] **[High]** Cross-boundary data flows documented and secured
- [ ] **[Critical]** No sensitive data in client-side storage (browser, mobile)

### 1.4 Network Architecture
- [ ] **[Critical]** Network segmentation: public, private, data tiers
- [ ] **[High]** Default deny network policies
- [ ] **[High]** Web Application Firewall on internet-facing endpoints
- [ ] **[Medium]** DDoS protection on public endpoints
- [ ] **[High]** No direct database access from public network
- [ ] **[High]** Egress filtering restricts outbound traffic
- [ ] **[Medium]** Service mesh for internal traffic policy enforcement

### 1.5 Third-Party Integration Security
- [ ] **[High]** All third-party APIs accessed over TLS
- [ ] **[High]** Third-party API keys stored in vault (not code or config)
- [ ] **[Medium]** Third-party API calls have timeouts and circuit breakers
- [ ] **[Medium]** Third-party data handling meets data classification requirements
- [ ] **[High]** Webhook endpoints validate sender authenticity (signatures)
- [ ] **[Medium]** Third-party vendor security assessment completed

---

## 2. Code Security Review

### 2.1 Input Validation
- [ ] **[Critical]** All user input validated on the server side
- [ ] **[Critical]** SQL injection prevented (parameterized queries, ORM usage)
- [ ] **[Critical]** XSS prevented (output encoding, CSP headers)
- [ ] **[Critical]** Command injection prevented (no shell execution with user input)
- [ ] **[Critical]** Path traversal prevented (no user input in file paths)
- [ ] **[High]** XML External Entity (XXE) prevented (if XML processing)
- [ ] **[High]** SSRF prevented (no user-controlled URLs for server-side requests)
- [ ] **[High]** Request body size limited
- [ ] **[High]** File upload validated: type, size, content, and stored outside webroot
- [ ] **[Medium]** Regular expressions validated for ReDoS vulnerability
- [ ] **[High]** Deserialization of untrusted data avoided or sandboxed

### 2.2 Authentication Code
- [ ] **[Critical]** Passwords hashed with bcrypt/Argon2id (cost factor >= 12)
- [ ] **[Critical]** No plaintext passwords in logs, errors, or responses
- [ ] **[Critical]** Timing-safe comparison for token/password verification
- [ ] **[High]** Rate limiting on login endpoints
- [ ] **[High]** Account lockout after repeated failures
- [ ] **[High]** Password reset tokens are single-use and time-limited
- [ ] **[Medium]** Login notification for new device/location

### 2.3 Session Management Code
- [ ] **[Critical]** Session IDs are cryptographically random (>= 128 bits)
- [ ] **[High]** Session invalidated on logout
- [ ] **[High]** Session invalidated on password change
- [ ] **[High]** Session timeout after inactivity
- [ ] **[Medium]** Concurrent session limits (if applicable)

### 2.4 Cryptography
- [ ] **[Critical]** No custom cryptographic implementations (use well-known libraries)
- [ ] **[Critical]** No deprecated algorithms (MD5, SHA1, DES, RC4)
- [ ] **[High]** Cryptographic keys generated with secure random number generators
- [ ] **[High]** Key management through KMS (not application configuration)
- [ ] **[Medium]** Cryptographic agility: algorithms can be updated without code changes

### 2.5 Error Handling
- [ ] **[High]** Internal error details not exposed to clients
- [ ] **[High]** Stack traces not returned in API responses
- [ ] **[High]** Database error messages not returned to clients
- [ ] **[Medium]** Custom error pages replace framework defaults
- [ ] **[High]** Errors logged with sufficient context for debugging

### 2.6 Dependency Security
- [ ] **[Critical]** No known critical CVEs in dependencies
- [ ] **[High]** No known high CVEs in dependencies
- [ ] **[High]** Dependencies pinned to exact versions
- [ ] **[High]** Lockfile present and committed
- [ ] **[Medium]** Unused dependencies removed
- [ ] **[High]** Automated vulnerability scanning in CI pipeline
- [ ] **[Medium]** Dependency update cadence defined and followed

### 2.7 Secrets Management
- [ ] **[Critical]** No secrets in source code (API keys, passwords, tokens)
- [ ] **[Critical]** No secrets in environment variables checked into version control
- [ ] **[High]** Secrets stored in vault (HashiCorp Vault, AWS Secrets Manager)
- [ ] **[High]** Secrets injected at runtime, not baked into images
- [ ] **[High]** Secret rotation supported without downtime
- [ ] **[Medium]** Secret access audited and logged

---

## 3. Infrastructure Security Review

### 3.1 Container Security
- [ ] **[High]** Base images from trusted registries
- [ ] **[High]** Minimal base images (distroless or Alpine)
- [ ] **[Critical]** Containers run as non-root user
- [ ] **[High]** Read-only root filesystem
- [ ] **[Critical]** No privileged containers
- [ ] **[High]** Resource limits defined (CPU, memory)
- [ ] **[High]** Image scanning in CI (fail on critical CVEs)
- [ ] **[Medium]** Image signing and verification
- [ ] **[Medium]** No unnecessary packages installed
- [ ] **[High]** Container health checks defined

### 3.2 Kubernetes Security
- [ ] **[High]** Pod Security Standards enforced (restricted profile)
- [ ] **[High]** RBAC with least-privilege service accounts
- [ ] **[High]** Network policies on all namespaces
- [ ] **[Critical]** Secrets encrypted at rest in etcd
- [ ] **[High]** Admission controllers enforce policies
- [ ] **[Medium]** Runtime security monitoring active
- [ ] **[Medium]** CIS Kubernetes benchmark compliance verified
- [ ] **[High]** No default service account tokens mounted
- [ ] **[Medium]** Pod-to-pod communication encrypted (mTLS via service mesh)

### 3.3 Cloud Infrastructure
- [ ] **[Critical]** IAM follows least privilege (no wildcard permissions)
- [ ] **[Critical]** No public S3 buckets / storage containers (unless intentional)
- [ ] **[High]** MFA on all human cloud console accounts
- [ ] **[High]** Cloud audit logging enabled (CloudTrail, Cloud Audit Logs)
- [ ] **[High]** Infrastructure defined as code (Terraform, Pulumi, CDK)
- [ ] **[Medium]** Drift detection active
- [ ] **[High]** Root account locked down (not used for daily operations)
- [ ] **[Medium]** Cost alerting configured (unexpected spend may indicate compromise)

### 3.4 Database Security
- [ ] **[Critical]** Database not accessible from public internet
- [ ] **[High]** Database connections encrypted (TLS)
- [ ] **[High]** Database authentication required (no anonymous access)
- [ ] **[High]** Database user per service (not shared credentials)
- [ ] **[High]** Database backups encrypted
- [ ] **[Medium]** Query auditing enabled for sensitive data access
- [ ] **[High]** Database patch level current

### 3.5 Certificate Management
- [ ] **[High]** TLS certificates auto-renewed (cert-manager, ACM)
- [ ] **[High]** Certificate expiry monitored and alerted
- [ ] **[Medium]** Certificate transparency logs monitored
- [ ] **[High]** Internal CA for mTLS certificates
- [ ] **[Medium]** OCSP stapling or CRL checking configured

---

## 4. Operational Security Review

### 4.1 Logging and Monitoring
- [ ] **[Critical]** Authentication events logged (success and failure)
- [ ] **[Critical]** Authorization failures logged
- [ ] **[High]** Administrative actions logged
- [ ] **[High]** Data access to restricted data logged
- [ ] **[High]** Logs shipped to centralized SIEM
- [ ] **[High]** Log integrity protected (tamper-evident storage)
- [ ] **[Critical]** No sensitive data in logs (PII, passwords, tokens)
- [ ] **[High]** Security alerts configured for anomalous patterns

### 4.2 Incident Response
- [ ] **[High]** Incident response plan documented
- [ ] **[High]** Incident response roles assigned
- [ ] **[Medium]** Incident response plan tested (tabletop exercise)
- [ ] **[High]** Breach notification procedure defined
- [ ] **[Medium]** Forensic evidence preservation procedure defined
- [ ] **[High]** Communication templates prepared

### 4.3 Business Continuity
- [ ] **[High]** Backup strategy tested (restore verified)
- [ ] **[High]** Recovery Time Objective (RTO) defined and achievable
- [ ] **[High]** Recovery Point Objective (RPO) defined and achievable
- [ ] **[Medium]** Disaster recovery plan documented and tested
- [ ] **[Medium]** Multi-region failover tested (if applicable)

### 4.4 Access Management
- [ ] **[High]** Employee onboarding grants minimum necessary access
- [ ] **[Critical]** Employee offboarding revokes all access within 24 hours
- [ ] **[High]** Access reviews conducted quarterly
- [ ] **[Medium]** Privileged access requires approval workflow
- [ ] **[High]** Shared accounts prohibited (individual accountability)

---

## 5. Compliance Verification

### 5.1 Data Privacy
- [ ] **[High]** Privacy policy published and accurate
- [ ] **[High]** Data processing inventory maintained
- [ ] **[High]** Consent management implemented (where required)
- [ ] **[High]** Data subject rights requests can be fulfilled (access, deletion, portability)
- [ ] **[Medium]** Data Processing Agreements with all sub-processors
- [ ] **[Medium]** Privacy Impact Assessment completed for new data processing

### 5.2 Regulatory
- [ ] **[High]** Applicable regulations identified and documented
- [ ] **[High]** Compliance controls mapped to regulatory requirements
- [ ] **[Medium]** Compliance evidence collection automated where possible
- [ ] **[High]** Regular compliance audits scheduled

---

## Audit Scoring

| Rating | Criteria |
|--------|----------|
| PASS | All Critical items pass, no more than 2 High items fail |
| CONDITIONAL PASS | All Critical items pass, 3-5 High items fail with remediation plan |
| FAIL | Any Critical item fails, or more than 5 High items fail |

---

*A security audit is not a one-time event. It is a periodic verification that the system's security posture has not degraded. Schedule audits quarterly and after any major architecture change.*
