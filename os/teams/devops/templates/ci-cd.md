# CI/CD Pipeline Template — SuperArchitect OS

**Version:** 1.0
**Owner:** DevOps Agent (Team 5)
**Last Updated:** 2026-03-29

---

## Purpose

This template defines the standard CI/CD pipeline for systems built by the SuperArchitect OS. It is platform-agnostic — adapt the syntax for your CI platform (GitHub Actions, GitLab CI, Jenkins, CircleCI) while preserving the stage structure and quality gates.

---

## 1. Pipeline Overview

```
Trigger: Push to main, PR creation, manual dispatch
         │
         ▼
┌─────────────────┐
│  1. Validate    │  Lint, format check, type check
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  2. Build       │  Compile, build artifact, container image
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  3. Unit Test   │  Fast tests, coverage report
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  4. Integration │  DB, broker, cache integration tests
│     Test        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  5. Security    │  SAST, SCA, container scan, secret scan
│     Scan        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  6. Deploy      │  Deploy to target environment
│     Staging     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  7. Smoke Test  │  Verify deployment health
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  8. Deploy      │  Production deployment (with approval)
│     Production  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  9. Post-Deploy │  Smoke tests, monitoring verification
│     Verify      │
└─────────────────┘
```

---

## 2. Stage Definitions

### Stage 1: Validate

**Purpose:** Catch syntax, style, and type errors before building.

```yaml
validate:
  steps:
    - name: Lint
      run: npm run lint  # or equivalent
      timeout: 2m

    - name: Format Check
      run: npm run format:check
      timeout: 1m

    - name: Type Check
      run: npm run typecheck
      timeout: 2m

    - name: Dependency Audit
      run: npm audit --audit-level=high
      timeout: 1m
```

**Gate:** All checks pass. Zero warnings treated as errors.

### Stage 2: Build

**Purpose:** Compile code and build deployable artifacts.

```yaml
build:
  steps:
    - name: Install Dependencies
      run: npm ci
      timeout: 3m

    - name: Compile
      run: npm run build
      timeout: 5m

    - name: Build Container Image
      run: |
        docker build -t $REGISTRY/$SERVICE:$GIT_SHA .
        docker tag $REGISTRY/$SERVICE:$GIT_SHA $REGISTRY/$SERVICE:$VERSION
      timeout: 5m

    - name: Push Image
      run: docker push $REGISTRY/$SERVICE:$GIT_SHA
      timeout: 3m
```

**Gate:** Build succeeds. Image pushed to registry.

### Stage 3: Unit Test

**Purpose:** Verify business logic correctness.

```yaml
unit-test:
  steps:
    - name: Run Unit Tests
      run: npm run test:unit -- --coverage --ci
      timeout: 5m

    - name: Upload Coverage Report
      run: upload-coverage ./coverage/lcov.info

    - name: Coverage Gate
      run: |
        COVERAGE=$(extract-coverage ./coverage/lcov.info)
        if [ "$COVERAGE" -lt 90 ]; then
          echo "Coverage $COVERAGE% below 90% threshold"
          exit 1
        fi
```

**Gate:** All tests pass. Coverage >= 90%.

### Stage 4: Integration Test

**Purpose:** Verify infrastructure integration works correctly.

```yaml
integration-test:
  services:
    postgres:
      image: postgres:16-alpine
      env: { POSTGRES_DB: test, POSTGRES_PASSWORD: test }
    redis:
      image: redis:7-alpine

  steps:
    - name: Run Migrations
      run: npm run db:migrate
      env: { DATABASE_URL: "postgresql://..." }
      timeout: 2m

    - name: Run Integration Tests
      run: npm run test:integration --ci
      timeout: 10m

    - name: Run Contract Tests
      run: npm run test:contract --ci
      timeout: 5m
```

**Gate:** All integration and contract tests pass.

### Stage 5: Security Scan

**Purpose:** Identify vulnerabilities before deployment.

```yaml
security-scan:
  parallel:
    - name: SAST Scan
      run: semgrep scan --config=auto --error
      timeout: 5m

    - name: Dependency Scan
      run: trivy fs --severity HIGH,CRITICAL --exit-code 1 .
      timeout: 3m

    - name: Container Scan
      run: trivy image --severity HIGH,CRITICAL --exit-code 1 $REGISTRY/$SERVICE:$GIT_SHA
      timeout: 5m

    - name: Secret Scan
      run: gitleaks detect --source . --exit-code 1
      timeout: 2m

    - name: IaC Scan
      run: checkov -d infrastructure/ --hard-fail-on HIGH,CRITICAL
      timeout: 3m
```

**Gate:** No critical or high findings. Medium findings logged as warnings.

### Stage 6: Deploy to Staging

**Purpose:** Deploy to staging environment for pre-production validation.

```yaml
deploy-staging:
  environment: staging
  steps:
    - name: Apply Infrastructure Changes
      run: |
        cd infrastructure/environments/staging
        terraform plan -out=tfplan
        terraform apply tfplan
      timeout: 10m

    - name: Run Database Migrations
      run: npm run db:migrate
      env: { DATABASE_URL: "$STAGING_DATABASE_URL" }
      timeout: 5m

    - name: Deploy Application
      run: |
        kubectl set image deployment/$SERVICE $SERVICE=$REGISTRY/$SERVICE:$GIT_SHA \
          --namespace=staging
        kubectl rollout status deployment/$SERVICE --namespace=staging --timeout=5m
      timeout: 7m
```

### Stage 7: Smoke Test

**Purpose:** Verify the staging deployment is functional.

```yaml
smoke-test:
  steps:
    - name: Health Check
      run: |
        curl -sf https://staging.example.com/health/ready || exit 1
      retries: 5
      timeout: 2m

    - name: API Smoke Tests
      run: npm run test:smoke -- --env=staging
      timeout: 5m

    - name: E2E Critical Paths
      run: npm run test:e2e -- --env=staging --suite=critical
      timeout: 10m
```

**Gate:** Health checks pass. Smoke tests pass. Critical E2E paths pass.

### Stage 8: Deploy to Production

**Purpose:** Deploy validated artifact to production.

```yaml
deploy-production:
  environment: production
  approval: required  # Manual approval for production
  steps:
    - name: Apply Infrastructure Changes
      run: |
        cd infrastructure/environments/production
        terraform plan -out=tfplan
        terraform apply tfplan
      timeout: 10m

    - name: Run Database Migrations
      run: npm run db:migrate
      env: { DATABASE_URL: "$PRODUCTION_DATABASE_URL" }
      timeout: 5m

    - name: Canary Deploy (10%)
      run: |
        kubectl-canary deploy $SERVICE $REGISTRY/$SERVICE:$GIT_SHA \
          --weight=10 --namespace=production
      timeout: 5m

    - name: Monitor Canary (5 min)
      run: |
        canary-monitor --service=$SERVICE --duration=5m \
          --error-threshold=1% --latency-threshold-p99=1000ms
      timeout: 7m

    - name: Promote to 100%
      run: |
        kubectl-canary promote $SERVICE --namespace=production
        kubectl rollout status deployment/$SERVICE --namespace=production --timeout=5m
      timeout: 7m
```

### Stage 9: Post-Deploy Verification

**Purpose:** Confirm production deployment is healthy.

```yaml
post-deploy-verify:
  steps:
    - name: Production Health Check
      run: |
        curl -sf https://api.example.com/health/ready || exit 1
      retries: 5
      timeout: 2m

    - name: Production Smoke Tests
      run: npm run test:smoke -- --env=production
      timeout: 5m

    - name: Verify Monitoring
      run: |
        check-metrics --service=$SERVICE --duration=5m \
          --error-rate-max=0.1% --latency-p99-max=1000ms
      timeout: 7m

    - name: Notify Success
      run: |
        notify-slack "#deployments" "✅ $SERVICE $VERSION deployed to production"
```

---

## 3. Branch Strategy

| Branch | Pipeline Stages | Deploy Target |
|--------|----------------|---------------|
| Feature branch (PR) | Validate, Build, Unit Test, Integration Test, Security Scan | None |
| `main` | All stages through Staging | Staging |
| Release tag | All stages through Production | Production |
| Hotfix branch | All stages (expedited) | Production |

---

## 4. Pipeline Configuration

### Environment Variables
```yaml
# Set per environment, never hardcoded
REGISTRY: container-registry.example.com
SERVICE: service-name
GIT_SHA: ${git rev-parse HEAD}
VERSION: ${semantic version from tag}
ENVIRONMENT: dev|staging|production
```

### Secrets (injected by CI platform)
```
DATABASE_URL
API_KEYS
REGISTRY_CREDENTIALS
CLOUD_CREDENTIALS
NOTIFICATION_WEBHOOKS
```

### Caching
```yaml
cache:
  - key: deps-${hashFiles('package-lock.json')}
    paths: [node_modules/]
  - key: docker-${hashFiles('Dockerfile')}
    paths: [/tmp/docker-cache/]
```

---

## 5. Failure Handling

### Automatic Rollback
```yaml
rollback:
  trigger: post-deploy health check failure OR error rate > 1% within 5 minutes
  steps:
    - name: Rollback Deployment
      run: kubectl rollout undo deployment/$SERVICE --namespace=production
    - name: Verify Rollback
      run: kubectl rollout status deployment/$SERVICE --namespace=production
    - name: Notify Failure
      run: notify-slack "#incidents" "🚨 $SERVICE deployment rolled back"
    - name: Create Incident
      run: create-incident --service=$SERVICE --type=failed-deployment
```

### Pipeline Failure Notifications
- Failed PR pipeline: Comment on PR with failure details
- Failed main pipeline: Notify team Slack channel
- Failed production deploy: Page on-call, create incident

---

## 6. Performance Budget

| Stage | Time Budget | Action if Exceeded |
|-------|------------|-------------------|
| Validate | 3 min | Optimize linter config |
| Build | 5 min | Optimize Dockerfile, caching |
| Unit Test | 5 min | Parallelize, optimize slow tests |
| Integration Test | 10 min | Parallelize, optimize fixtures |
| Security Scan | 5 min | Parallelize scanners |
| Deploy | 10 min | Optimize rollout strategy |
| Total Pipeline | 30 min | Mandatory optimization sprint |

---

*A CI/CD pipeline is the factory floor of software delivery. Keep it fast, reliable, and observable. A slow or flaky pipeline is a tax on every engineer, every day.*
