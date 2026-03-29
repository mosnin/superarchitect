# Infrastructure Template — SuperArchitect OS

**Version:** 1.0
**Owner:** DevOps Agent (Team 5)
**Last Updated:** 2026-03-29

---

## Purpose

This template defines the standard infrastructure architecture for production systems. It is cloud-agnostic in design principles but includes AWS/GCP/Azure mapping for common components. Adapt to your cloud provider while preserving the architectural requirements.

---

## 1. Infrastructure Architecture

### 1.1 Network Topology

```
┌─────────────────────────────────────────────────────────┐
│                        VPC / VNet                        │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Public Subnet (DMZ)                 │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │    │
│  │  │   ALB    │  │   NAT    │  │   Bastion    │  │    │
│  │  │          │  │  Gateway │  │   (if needed) │  │    │
│  │  └──────────┘  └──────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │            Private Subnet (Application)          │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │    │
│  │  │  K8s     │  │  K8s     │  │  K8s         │  │    │
│  │  │  Node 1  │  │  Node 2  │  │  Node 3      │  │    │
│  │  └──────────┘  └──────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │            Isolated Subnet (Data)                │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │    │
│  │  │ Database │  │  Cache   │  │  Message     │  │    │
│  │  │ Primary  │  │  Cluster │  │  Broker      │  │    │
│  │  └──────────┘  └──────────┘  └──────────────┘  │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Network Requirements
- VPC with at least 3 availability zones
- Public subnet: only load balancers, NAT gateways, and bastion hosts
- Private subnet: application workloads (Kubernetes nodes, compute instances)
- Isolated subnet: databases, caches, message brokers (no internet access)
- NAT gateway for outbound internet access from private subnets
- VPC flow logs enabled for network troubleshooting and security

### 1.3 DNS
- External DNS: Route53, Cloud DNS, or Azure DNS
- Internal DNS: Kubernetes CoreDNS for service discovery
- DNS TTL: 60 seconds for services (fast failover), 3600 for static records
- Health check-based DNS failover for multi-region deployments

---

## 2. Compute Infrastructure

### 2.1 Kubernetes Cluster

**Cluster Configuration:**
```hcl
resource "kubernetes_cluster" "main" {
  name               = "${var.project}-${var.environment}"
  kubernetes_version = "1.29"  # Stay within N-1 of latest

  # Control plane
  control_plane {
    high_availability = true  # Multi-AZ control plane
  }

  # Node pools
  node_pool "general" {
    instance_type  = "m6i.xlarge"    # 4 vCPU, 16GB RAM
    min_nodes      = 3
    max_nodes      = 20
    disk_size_gb   = 100
    labels         = { workload = "general" }
  }

  node_pool "compute" {
    instance_type  = "c6i.2xlarge"   # 8 vCPU, 16GB RAM
    min_nodes      = 0
    max_nodes      = 10
    labels         = { workload = "compute-intensive" }
    taints         = [{ key = "workload", value = "compute", effect = "NoSchedule" }]
  }

  # Cluster add-ons
  addons = [
    "metrics-server",
    "cluster-autoscaler",
    "aws-load-balancer-controller",
    "cert-manager",
    "external-dns",
  ]
}
```

### 2.2 Node Pool Strategy
| Pool | Instance Type | Use Case | Scaling |
|------|--------------|----------|---------|
| General | Balanced (m-series) | API services, web servers | 3-20 nodes |
| Compute | CPU-optimized (c-series) | Data processing, ML inference | 0-10 nodes |
| Memory | Memory-optimized (r-series) | Caching, in-memory processing | 0-5 nodes |
| Spot/Preemptible | Mixed types | Batch jobs, non-critical workloads | 0-20 nodes |

---

## 3. Data Infrastructure

### 3.1 Primary Database

```hcl
resource "database_cluster" "primary" {
  engine           = "postgresql"
  engine_version   = "16"
  instance_class   = "db.r6g.xlarge"   # Size per workload analysis

  # High availability
  multi_az             = true
  read_replicas        = 2
  backup_retention_days = 30

  # Security
  encryption_at_rest   = true
  encryption_key       = aws_kms_key.database.arn
  publicly_accessible  = false
  subnet_group         = aws_db_subnet_group.isolated.id

  # Performance
  iops                 = 3000
  storage_type         = "io2"
  max_allocated_storage = 1000  # Auto-scaling storage

  # Monitoring
  performance_insights = true
  monitoring_interval  = 60
  enhanced_monitoring  = true
}
```

### 3.2 Cache Layer

```hcl
resource "cache_cluster" "redis" {
  engine             = "redis"
  engine_version     = "7.0"
  node_type          = "cache.r6g.large"
  num_cache_nodes    = 3               # Cluster mode
  automatic_failover = true

  # Security
  at_rest_encryption = true
  in_transit_encryption = true
  auth_token         = var.redis_auth_token
  subnet_group       = aws_elasticache_subnet_group.isolated.id

  # Maintenance
  snapshot_retention = 7
  maintenance_window = "sun:05:00-sun:06:00"
}
```

### 3.3 Message Broker

```hcl
resource "message_broker" "kafka" {
  broker_count       = 3
  instance_type      = "kafka.m5.large"
  ebs_volume_size    = 500

  # Security
  encryption_in_transit = "TLS"
  encryption_at_rest    = true

  # Configuration
  kafka_version      = "3.6"
  configuration {
    auto_create_topics = false       # Topics created via IaC only
    default_replication_factor = 3
    min_insync_replicas = 2
    log_retention_hours = 168        # 7 days
  }

  # Monitoring
  enhanced_monitoring = "PER_TOPIC_PER_BROKER"
}
```

---

## 4. Observability Infrastructure

### 4.1 Monitoring Stack

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Prometheus  │────>│   Grafana    │     │   Alerting   │
│  (Metrics)   │     │ (Dashboards) │     │ (PagerDuty)  │
└──────────────┘     └──────────────┘     └──────────────┘
       │
┌──────────────┐     ┌──────────────┐
│  Loki/ELK   │────>│   Grafana    │
│  (Logs)      │     │ (Log Search) │
└──────────────┘     └──────────────┘
       │
┌──────────────┐     ┌──────────────┐
│   Tempo/     │────>│   Grafana    │
│   Jaeger     │     │  (Traces)    │
│  (Tracing)   │     │              │
└──────────────┘     └──────────────┘
```

### 4.2 Monitoring Components
| Component | Purpose | Retention |
|-----------|---------|-----------|
| Prometheus | Metrics collection and alerting | 30 days (local), 1 year (long-term) |
| Grafana | Dashboards and visualization | N/A (config in IaC) |
| Loki or ELK | Log aggregation and search | 90 days |
| Tempo or Jaeger | Distributed tracing | 7 days |
| Alertmanager | Alert routing and deduplication | N/A |
| PagerDuty/Opsgenie | On-call management and escalation | N/A |

---

## 5. Security Infrastructure

### 5.1 WAF Rules
```hcl
resource "waf_acl" "main" {
  rules = [
    { name = "rate-limit",      priority = 1, action = "block", rate_limit = 2000 },
    { name = "ip-reputation",   priority = 2, action = "block", managed_rule = "ip-reputation" },
    { name = "common-exploits", priority = 3, action = "block", managed_rule = "common-exploits" },
    { name = "sqli",            priority = 4, action = "block", managed_rule = "sqli" },
    { name = "xss",             priority = 5, action = "block", managed_rule = "xss" },
    { name = "bot-control",     priority = 6, action = "challenge", managed_rule = "bot-control" },
  ]
}
```

### 5.2 Secrets Management
```hcl
resource "vault_mount" "secrets" {
  path = "secret"
  type = "kv-v2"
}

# External Secrets Operator syncs vault secrets to Kubernetes
resource "helm_release" "external_secrets" {
  name       = "external-secrets"
  repository = "https://charts.external-secrets.io"
  chart      = "external-secrets"
}
```

### 5.3 Certificate Management
- cert-manager for automated TLS certificate lifecycle
- Let's Encrypt for external certificates
- Internal CA (Vault PKI) for mTLS certificates
- Certificate rotation: external (90 days), internal (30 days)

---

## 6. Backup and Disaster Recovery

### 6.1 Backup Strategy
| Resource | Frequency | Retention | Location |
|----------|-----------|-----------|----------|
| Database | Continuous (WAL) + daily snapshot | 30 days | Cross-region |
| Object storage | Versioning + cross-region replication | 90 days | Different region |
| Kubernetes state | etcd snapshot daily | 30 days | External storage |
| Configuration | Git (version controlled) | Forever | Git remote |

### 6.2 Recovery Objectives
| Tier | RTO | RPO | Example |
|------|-----|-----|---------|
| Tier 1 (Critical) | 1 hour | 5 minutes | Database, auth service |
| Tier 2 (Important) | 4 hours | 1 hour | API services, workers |
| Tier 3 (Standard) | 24 hours | 24 hours | Analytics, reports |

### 6.3 DR Testing
- Backup restore tested monthly
- Failover tested quarterly
- Full DR drill annually
- Results documented and deficiencies tracked to remediation

---

## 7. Cost Management

### 7.1 Cost Controls
- Resource tagging mandatory: environment, team, service, cost-center
- Budget alerts at 50%, 80%, 100% of expected spend
- Right-sizing reviews monthly (check utilization vs. allocation)
- Reserved instances for stable baseline workloads
- Spot instances for fault-tolerant batch workloads
- Auto-scaling to match actual demand

### 7.2 Cost Optimization Checklist
- [ ] Unused resources identified and removed
- [ ] Oversized instances right-sized
- [ ] Storage tier matches access patterns (hot/warm/cold)
- [ ] Data transfer costs minimized (VPC endpoints, regional affinity)
- [ ] Reserved capacity purchased for predictable workloads
- [ ] Dev/staging environments scaled down outside business hours

---

*Infrastructure decisions made today determine operational costs and reliability for years. Design for the load you will have in 12 months, not the load you have today. But also do not over-provision — cloud resources are elastic by design.*
