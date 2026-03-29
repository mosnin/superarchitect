# DevOps Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** DevOps Agent (Team 5)
**Last Updated:** 2026-03-29

---

## Purpose

These standards define infrastructure, deployment, and operational requirements for every system built by the SuperArchitect OS. Infrastructure is code. Deployments are automated. Operations are observable. Exceptions require documented justification.

---

## 1. Infrastructure as Code Standards

### 1.1 Core Principles
- All infrastructure is defined in code (Terraform, Pulumi, CDK, or equivalent)
- No manual changes to any environment (dev, staging, production)
- Infrastructure code is versioned, reviewed, and tested like application code
- State files are stored remotely with locking (S3 + DynamoDB, GCS + Cloud Storage)
- Drift detection runs daily; drift is treated as a defect

### 1.2 Code Organization
```
infrastructure/
  modules/           # Reusable infrastructure modules
    networking/
    compute/
    database/
    monitoring/
    security/
  environments/
    dev/             # Dev environment composition
    staging/         # Staging environment composition
    production/      # Production environment composition
  policies/          # OPA/Sentinel policy files
  tests/             # Infrastructure tests
```

### 1.3 Module Standards
- Every module has a README with inputs, outputs, and usage example
- Variables have descriptions, types, and validation rules
- Outputs are documented and stable (changing outputs is a breaking change)
- Modules are versioned (semantic versioning via git tags)
- No hardcoded values: region, account ID, environment name are all variables

### 1.4 Security in IaC
- No secrets in IaC code or state files
- IAM policies follow least privilege
- Security groups/network policies default to deny
- Encryption enabled by default on all resources
- Policy-as-code validation in CI (OPA, Sentinel, Checkov)

---

## 2. Container Standards

### 2.1 Dockerfile Best Practices
```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Minimal runtime image
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER nonroot
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD ["node", "dist/health.js"]
CMD ["dist/server.js"]
```

### 2.2 Container Rules
- Multi-stage builds: separate build and runtime stages
- Minimal base images: distroless preferred, Alpine acceptable
- Non-root user (UID > 1000)
- Read-only root filesystem (mount writable volumes for temp/logs)
- No secrets in images (no ARG/ENV for passwords)
- Resource limits defined in orchestrator (not unlimited)
- Health checks defined in Dockerfile and orchestrator
- Labels for metadata: version, commit SHA, build date
- `.dockerignore` excludes: `.git`, `node_modules`, `.env`, tests

### 2.3 Image Management
- Images tagged with: git SHA (immutable) and semantic version
- `latest` tag exists only in development
- Image scanning in CI: fail on critical/high CVEs
- Image registry uses private repositories
- Image retention policy: keep last 30 tagged images, prune untagged
- Image signing for production deployments (cosign, Notary)

---

## 3. Kubernetes Standards

### 3.1 Resource Definitions

Every service deployment includes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-name
  labels:
    app: service-name
    version: "1.2.3"
    team: team-name
spec:
  replicas: 2                    # Minimum 2 for HA
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0          # Zero-downtime deploys
  template:
    spec:
      serviceAccountName: service-name  # Dedicated service account
      securityContext:
        runAsNonRoot: true
        fsGroup: 1000
      containers:
      - name: service-name
        image: registry/service:sha-abc123
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 15
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        startupProbe:
          httpGet:
            path: /health/startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 2
```

### 3.2 Required Kubernetes Resources per Service
- Deployment with rolling update strategy
- Service (ClusterIP for internal, LoadBalancer/Ingress for external)
- HorizontalPodAutoscaler (CPU and/or custom metrics)
- PodDisruptionBudget (minAvailable >= 1)
- NetworkPolicy (restrict ingress and egress)
- ServiceAccount (dedicated, no default token mount)
- ConfigMap for non-secret configuration
- ExternalSecret or SealedSecret for secrets

### 3.3 Namespace Strategy
- One namespace per team or per application domain
- Resource quotas on every namespace
- Limit ranges for default resource requests/limits
- Network policies default deny in every namespace
- RBAC: teams have access only to their namespaces

### 3.4 Scaling
- HPA configured with both CPU and memory targets
- Custom metrics scaling for queue-based or event-driven services
- Cluster autoscaler enabled with appropriate min/max node counts
- Vertical Pod Autoscaler in recommendation mode (not auto-update)
- Pod topology spread constraints for high availability

---

## 4. CI/CD Standards

### 4.1 Pipeline Architecture

```
┌─────────┐   ┌──────────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐
│  Commit  │──>│   Build  │──>│   Test   │──>│  Security │──>│  Deploy  │
└─────────┘   └──────────┘   └──────────┘   └───────────┘   └──────────┘
                                                                  │
                                                           ┌──────────┐
                                                           │  Verify  │
                                                           └──────────┘
```

### 4.2 Pipeline Stages

**Stage 1: Build**
- Compile/transpile source code
- Build container image
- Tag image with git SHA
- Push to registry
- Time budget: < 5 minutes

**Stage 2: Test**
- Run unit tests with coverage reporting
- Run integration tests (with testcontainers)
- Run contract tests
- Coverage gate: fail if below threshold
- Time budget: < 10 minutes

**Stage 3: Security**
- SAST scan (Semgrep, SonarQube)
- Dependency vulnerability scan (Snyk, Trivy)
- Container image scan
- Secret scan (gitleaks, trufflehog)
- IaC policy scan (Checkov, tfsec)
- Gate: fail on critical/high findings
- Time budget: < 5 minutes

**Stage 4: Deploy**
- Deploy to target environment
- Run database migrations
- Apply infrastructure changes
- Time budget: < 10 minutes

**Stage 5: Verify**
- Smoke tests against deployed environment
- Health check verification
- Monitoring verification (metrics flowing, logs appearing)
- Time budget: < 5 minutes

### 4.3 Pipeline Rules
- Total pipeline time: < 30 minutes end-to-end
- Pipeline-as-code (Jenkinsfile, .github/workflows, .gitlab-ci.yml)
- Pipeline changes go through code review
- No manual approval gates for dev/staging (production optional)
- Artifacts are immutable: same artifact deploys to all environments
- Environment-specific configuration injected at deploy time

### 4.4 Deployment Strategies

| Strategy | Use Case | Rollback Speed |
|----------|----------|---------------|
| Rolling update | Default for all services | 1-5 minutes |
| Blue-green | Database schema changes, high-risk releases | Instant |
| Canary | User-facing features, uncertain impact | 1-2 minutes |
| Feature flags | Gradual feature rollout | Instant |

### 4.5 Rollback Requirements
- Rollback to previous version in < 5 minutes
- Automated rollback on health check failure
- Database migration rollback scripts tested
- Rollback procedure documented and practiced monthly

---

## 5. Observability Standards

### 5.1 Logging

**Standard:** All logs are structured JSON.

Required fields per log entry:
```json
{
  "timestamp": "ISO-8601",
  "level": "INFO|WARN|ERROR|DEBUG",
  "service": "service-name",
  "version": "1.2.3",
  "environment": "production",
  "traceId": "w3c-trace-id",
  "spanId": "span-id",
  "message": "Human-readable message",
  "context": {}
}
```

Log level guidelines:
- **ERROR**: Requires human attention. Alert may fire.
- **WARN**: Unexpected but handled. Investigate if frequent.
- **INFO**: Significant business events. Audit trail.
- **DEBUG**: Diagnostic detail. Off in production unless troubleshooting.

### 5.2 Metrics

**Standard:** Prometheus-compatible metrics endpoint (`/metrics`).

Required metrics per service:
- `http_requests_total` (counter: method, path, status)
- `http_request_duration_seconds` (histogram: method, path)
- `http_requests_in_flight` (gauge)
- `db_query_duration_seconds` (histogram: query)
- `db_connections_active` (gauge)
- `external_request_duration_seconds` (histogram: service, method)
- `circuit_breaker_state` (gauge: service, state)
- Custom business metrics per domain

### 5.3 Distributed Tracing
- W3C Trace Context propagation on all communications
- Spans for: HTTP handlers, database queries, cache operations, message publish/consume, external API calls
- Trace sampling: 100% for errors, configurable for normal traffic (default 10%)
- Trace data retained for minimum 7 days

### 5.4 Alerting
- Alert on symptoms (user-facing impact), not causes
- Every alert has a runbook link
- Alert severity: Critical (page immediately), Warning (investigate within 4h), Info (review next business day)
- Alert fatigue prevention: tune thresholds, suppress duplicates, require action for every alert
- On-call rotation with escalation policy

### 5.5 Dashboards
Required dashboards per service:
1. **Service overview**: RED metrics, error rate, saturation
2. **Infrastructure**: CPU, memory, disk, network
3. **Dependencies**: Health and latency of downstream services
4. **Business**: Domain-specific metrics and KPIs

---

## 6. Runbook Standards

### 6.1 Runbook Template
Every service must have a runbook covering:

1. **Service Overview**: What it does, who owns it, dependencies
2. **Architecture**: Deployment topology, data stores, integrations
3. **Common Operations**: Scale up/down, restart, configuration changes
4. **Troubleshooting**: Symptoms, likely causes, diagnostic commands, resolution steps
5. **Incident Response**: Escalation contacts, communication templates
6. **Recovery Procedures**: Restore from backup, failover, data recovery

### 6.2 Runbook Rules
- Runbooks live in the repository alongside the service code
- Runbooks are updated as part of every operational change
- Runbooks are tested (new team member follows the runbook to verify clarity)
- Runbook links appear in alert notifications
- Runbooks never assume prior knowledge of the system

---

## 7. Environment Standards

### 7.1 Environment Parity
- Staging is architecturally identical to production
- Same container images deploy to all environments
- Configuration differences are minimal and documented
- Data volume in staging >= 10% of production (for realistic testing)

### 7.2 Environment Lifecycle
| Environment | Purpose | Data | Stability |
|-------------|---------|------|-----------|
| Local dev | Developer iteration | Synthetic | Ephemeral |
| CI | Automated testing | Test fixtures | Ephemeral |
| Staging | Pre-production validation | Anonymized production mirror | Stable |
| Production | Live traffic | Real | Highly stable |

---

*Infrastructure is not an afterthought. It is the foundation that determines whether your application survives contact with reality. Treat it with the same rigor as application code.*
