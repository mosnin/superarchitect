# Security Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** Security Agent (Team 6)
**Last Updated:** 2026-03-29

---

## Purpose

These standards define the security baseline for every system built by the SuperArchitect OS. Security is not a feature — it is a structural property. These standards are mandatory from the first line of code and enforced throughout the system lifecycle.

---

## 1. Authentication Standards

### 1.1 Identity Provider
- Use a dedicated identity provider (Auth0, Cognito, Keycloak, or equivalent)
- Never build custom authentication from scratch unless required by regulatory constraints
- Support multi-factor authentication (MFA) for all human users
- Enforce MFA for administrative and privileged accounts

### 1.2 Token Management
- Use short-lived access tokens (15-30 minutes)
- Use refresh tokens with rotation (single-use refresh tokens)
- Store tokens securely: HTTP-only, Secure, SameSite=Strict cookies for web; secure storage for mobile
- Never store tokens in localStorage or sessionStorage
- Token validation must check: signature, expiry, issuer, audience, and scope

### 1.3 Password Policy (if applicable)
- Minimum 12 characters
- No maximum length below 128 characters
- Check against breached password databases (Have I Been Pwned API)
- Hash with bcrypt (cost factor >= 12), Argon2id, or scrypt
- Never store passwords in plaintext or reversible encryption
- Rate limit authentication attempts (5 failures per 15 minutes per account)
- Account lockout after 10 consecutive failures with notification

### 1.4 Session Management
- Session IDs must be cryptographically random (minimum 128 bits)
- Sessions expire after inactivity (default: 30 minutes)
- Session invalidation on password change, privilege change, or explicit logout
- Bind sessions to user agent and IP range (optional, configurable)

### 1.5 Service-to-Service Authentication
- mTLS for all internal service communication
- Service accounts with unique credentials per service
- API keys for machine-to-machine integrations (rotated quarterly)
- OAuth 2.0 client credentials flow for service-to-service authorization

---

## 2. Authorization Standards

### 2.1 Access Control Model
- Implement Role-Based Access Control (RBAC) as the minimum
- Attribute-Based Access Control (ABAC) for fine-grained requirements
- Policy decision point must be centralized (OPA, Cedar, or equivalent)
- Policy enforcement at every service boundary (never trust upstream authorization)

### 2.2 Principle of Least Privilege
- Every user, service, and process gets the minimum permissions required
- Default deny: access is denied unless explicitly granted
- Permissions are granular: read, write, delete, admin per resource type
- Elevated permissions require explicit approval and time-boxing

### 2.3 Resource-Level Authorization
- Users can only access resources they own or have been granted access to
- Every data access query includes an authorization filter
- Object-level authorization is verified on every request (not just collection-level)
- Insecure Direct Object Reference (IDOR) prevention on all endpoints

### 2.4 Administrative Access
- Admin interfaces are network-isolated (not on the public internet)
- Admin actions require MFA step-up authentication
- All admin actions are logged with full audit trail
- Break-glass procedures documented for emergency access
- Admin credentials rotated regularly (minimum quarterly)

---

## 3. Data Security Standards

### 3.1 Data Classification

| Level | Description | Examples | Controls |
|-------|-------------|----------|----------|
| Public | Intentionally public | Marketing content, docs | Integrity protection |
| Internal | Not for external disclosure | Employee directories | Access control, logging |
| Confidential | Business-sensitive | Financial data, strategies | Encryption, strict access |
| Restricted | Regulated or high-impact | PII, PHI, payment data | Encryption, audit, retention |

### 3.2 Encryption at Rest
- All databases encrypted at rest (AES-256 or equivalent)
- All file storage encrypted at rest
- All backups encrypted
- Encryption keys managed by a KMS (AWS KMS, GCP KMS, HashiCorp Vault)
- Key rotation policy: annual minimum, immediate on compromise
- Application-level encryption for Restricted data (envelope encryption)

### 3.3 Encryption in Transit
- TLS 1.2 minimum for all external communications (TLS 1.3 preferred)
- mTLS for all internal service-to-service communication
- Certificate management automated (cert-manager, ACM, Let's Encrypt)
- HSTS headers on all web responses (max-age >= 1 year, includeSubDomains)
- Certificate pinning for mobile applications

### 3.4 Data Handling
- PII minimization: collect only what is needed, retain only as long as required
- Data masking in non-production environments
- Data anonymization for analytics and reporting
- Right to deletion (GDPR Article 17) implemented and tested
- Data retention policies defined and automated per data classification

---

## 4. API Security Standards

### 4.1 Input Validation
- Validate all input on the server side (never trust client validation)
- Use allowlists, not denylists, for input validation
- Validate: type, length, range, format, and allowed characters
- Reject unexpected fields (strict schema validation)
- Limit request body size (default: 1MB, configurable per endpoint)
- Limit array/list sizes in request bodies
- Limit nested object depth (default: 5 levels)

### 4.2 Output Encoding
- Encode all output for the appropriate context (HTML, JSON, URL, SQL)
- Never render raw user input in responses
- Content-Type headers set correctly on all responses
- X-Content-Type-Options: nosniff on all responses
- Content-Security-Policy headers on all web responses

### 4.3 Rate Limiting
- Rate limiting on all public endpoints
- Stricter rate limiting on authentication endpoints (login, register, password reset)
- Per-client rate limiting (API key, IP, user ID)
- Return 429 with Retry-After header when rate limited
- Rate limit state stored externally (Redis) for distributed deployments

### 4.4 CORS Policy
- CORS origins explicitly allowlisted (never use wildcard `*` with credentials)
- Allowed methods restricted to those actually used
- Allowed headers restricted to those actually needed
- Preflight responses cached (Access-Control-Max-Age)
- CORS configuration reviewed and tested

### 4.5 API Keys and Secrets
- API keys transmitted in headers, never in URLs (URLs are logged)
- API keys are cryptographically random (minimum 256 bits)
- API key rotation supported without downtime
- Compromised keys revocable immediately
- Key scoping: each key has defined permissions and rate limits

---

## 5. Infrastructure Security Standards

### 5.1 Network Security
- Default deny network policies (zero-trust networking)
- Service mesh for internal traffic encryption and policy enforcement
- Web Application Firewall (WAF) on all internet-facing endpoints
- DDoS protection on all public endpoints
- Network segmentation: public, private, and data tiers
- Bastion hosts or VPN for administrative access (no direct SSH)
- Egress filtering: restrict outbound traffic to known destinations

### 5.2 Container Security
- Base images from trusted registries only
- Minimal base images (distroless or Alpine)
- Non-root user in containers
- Read-only root filesystem
- No privileged containers
- Resource limits (CPU, memory) on all containers
- Image scanning in CI pipeline (fail on critical/high CVEs)
- Image signing and verification

### 5.3 Kubernetes Security
- Pod Security Standards enforced (restricted profile)
- RBAC with least-privilege service accounts
- Network policies on all namespaces
- Secrets encrypted at rest in etcd
- Admission controllers for policy enforcement (OPA Gatekeeper, Kyverno)
- Runtime security monitoring (Falco or equivalent)
- Regular CIS Kubernetes benchmark audits

### 5.4 Cloud IAM
- Least privilege IAM policies for all services
- No wildcard permissions (no `*` in Action or Resource)
- IAM roles over IAM users for service accounts
- MFA required for all human cloud console access
- Regular IAM access reviews (quarterly minimum)
- Unused permissions and accounts removed promptly

---

## 6. Logging and Monitoring Security

### 6.1 Security Logging Requirements
Log the following events at minimum:
- Authentication success and failure
- Authorization failures
- Input validation failures
- Privilege escalation events
- Data access to Restricted classification data
- Administrative actions
- Configuration changes
- Security-relevant system events (certificate rotation, key rotation)

### 6.2 Log Protection
- Logs stored in append-only, tamper-evident storage
- Log access restricted to authorized personnel
- No sensitive data in logs (passwords, tokens, PII, credit card numbers)
- Log retention meets compliance requirements (minimum 90 days, 1 year for regulated)
- Logs shipped to centralized SIEM for correlation and analysis

### 6.3 Security Alerting
| Event | Severity | Response |
|-------|----------|----------|
| Brute force authentication attempts | High | Auto-block, notify security |
| Privilege escalation attempt | Critical | Page security team |
| Unusual data access patterns | Medium | Investigate within 4 hours |
| Dependency vulnerability (critical) | High | Patch within 24 hours |
| Certificate expiring (< 30 days) | Warning | Renew immediately |
| Security scan failure in CI | High | Block deployment |

---

## 7. Dependency Security

### 7.1 Dependency Management
- Pin all dependency versions (exact versions, not ranges)
- Automated dependency vulnerability scanning in CI (Snyk, Dependabot, Renovate)
- Critical vulnerabilities block the build
- High vulnerabilities require remediation plan within 48 hours
- Dependency updates reviewed and tested before merging
- Transitive dependencies scanned (not just direct dependencies)

### 7.2 Supply Chain Security
- Use lockfiles (package-lock.json, Cargo.lock, go.sum)
- Verify package integrity (checksums, signatures)
- Use private package registries for internal packages
- Limit the number of maintainers for internal packages
- Monitor for typosquatting attacks on package names

---

## 8. Compliance Frameworks

### 8.1 SOC 2 Type II
Required controls:
- Access control and authentication (CC6.1-CC6.8)
- Change management (CC8.1)
- Risk assessment (CC3.1-CC3.4)
- Monitoring and logging (CC7.1-CC7.4)
- Incident response (CC7.3-CC7.5)
- Data encryption (CC6.7)
- Vendor management (CC9.2)

### 8.2 GDPR
Required capabilities:
- Consent management and tracking
- Data subject access requests (DSAR) fulfillment
- Right to erasure implementation
- Data portability (export in machine-readable format)
- Breach notification process (72-hour window)
- Data Processing Agreements with all sub-processors
- Privacy Impact Assessments for new data processing

### 8.3 HIPAA (if handling PHI)
Required controls:
- Business Associate Agreements with all vendors
- PHI access logging and audit trail
- PHI encryption at rest and in transit
- Minimum necessary access principle
- Workforce training and awareness
- Incident response and breach notification
- Regular risk assessments

---

## 9. Vulnerability Management

### 9.1 Severity SLAs

| Severity | Remediation SLA | Verification |
|----------|----------------|-------------|
| Critical | 24 hours | Immediate rescan |
| High | 7 days | Next CI run |
| Medium | 30 days | Next release |
| Low | 90 days | Quarterly review |

### 9.2 Vulnerability Response Process
1. Triage: Confirm vulnerability and assess impact
2. Prioritize: Assign severity based on CVSS + business context
3. Remediate: Patch, mitigate, or accept with documented risk
4. Verify: Confirm fix eliminates the vulnerability
5. Communicate: Notify affected stakeholders
6. Prevent: Update standards to prevent recurrence

---

*Security is not a destination. It is a continuous process of identifying, mitigating, and monitoring risks. These standards are the baseline — not the ceiling.*
