# Data Engineering Standards — SuperArchitect OS

**Version:** 1.0
**Owner:** Data Agent (Team 8)
**Last Updated:** 2026-03-29

---

## Purpose

These standards define the baseline for data engineering work within the SuperArchitect OS. They cover data modeling, pipeline design, data quality, governance, and operational requirements. Every data system must meet these standards before production use.

---

## 1. Data Modeling Standards

### 1.1 Relational Data Modeling
- Normalize to 3NF for transactional systems (denormalize only with documented justification)
- Every table has a primary key (prefer UUID or ULID for distributed systems)
- Foreign keys enforced at the database level (application-level only is insufficient)
- Timestamps on every table: `created_at`, `updated_at` (UTC, TIMESTAMPTZ)
- Soft delete via `deleted_at` column (hard delete only with data retention policy approval)
- Column naming: snake_case, descriptive, no abbreviations
- Table naming: snake_case, plural nouns (e.g., `orders`, `order_items`)
- Enum values stored as strings (not integers) for readability
- JSON columns used sparingly — only for truly schema-less data

### 1.2 Schema Evolution
- All schema changes via versioned migration scripts
- Migrations must be backward-compatible (support N-1 application version)
- Large table migrations use online DDL (pt-online-schema-change, gh-ost, or native)
- Rollback migration exists for every forward migration
- Migration testing includes: forward, rollback, forward again (idempotency)
- No data loss migrations without explicit approval and backup verification

### 1.3 Indexing Strategy
- Every foreign key column is indexed
- Every column used in WHERE, JOIN, or ORDER BY is evaluated for indexing
- Composite indexes ordered by selectivity (most selective column first)
- Index usage verified with EXPLAIN ANALYZE on production-like data volume
- Unused indexes identified and removed quarterly
- Partial indexes used where applicable (e.g., `WHERE status = 'active'`)

### 1.4 Analytical Data Modeling
- Star schema or snowflake schema for data warehouses
- Fact tables contain measures; dimension tables contain descriptors
- Slowly Changing Dimensions (SCD) strategy defined per dimension (Type 1, 2, or 3)
- Grain of each fact table documented explicitly
- Conformed dimensions shared across fact tables
- Date dimension table with pre-computed calendar attributes

---

## 2. Data Pipeline Standards

### 2.1 Pipeline Design Principles
- Pipelines are idempotent: running the same pipeline twice produces the same result
- Pipelines are atomic: they succeed completely or fail completely (no partial updates)
- Pipelines are observable: every run produces metrics, logs, and status
- Pipelines are testable: unit tests for transformations, integration tests for end-to-end
- Pipelines handle late-arriving data gracefully

### 2.2 ETL/ELT Architecture

**Preferred: ELT (Extract, Load, Transform)**
1. **Extract**: Pull data from source systems with minimal transformation
2. **Load**: Land raw data in the staging area (data lake or staging schema)
3. **Transform**: Apply business logic in the warehouse/lakehouse

**When ETL is necessary:**
- Source data volume exceeds warehouse capacity
- Data must be filtered/anonymized before landing
- Real-time transformation is required

### 2.3 Pipeline Orchestration
- Use a workflow orchestrator: Airflow, Dagster, Prefect, or equivalent
- DAGs define pipeline dependencies explicitly
- Retry logic with exponential backoff on transient failures
- Alerting on pipeline failure, SLA miss, and data quality violation
- Pipeline runs have unique identifiers for traceability
- Backfill capability for all pipelines (reprocess historical data)

### 2.4 Pipeline Patterns

| Pattern | Use Case | Latency | Complexity |
|---------|----------|---------|------------|
| Batch | Daily/hourly aggregations, reports | Hours | Low |
| Micro-batch | Near-real-time analytics (5-15 min windows) | Minutes | Medium |
| Streaming | Real-time dashboards, alerts, event processing | Seconds | High |
| CDC | Database change capture, event sourcing | Seconds-minutes | Medium |
| Lambda | Batch accuracy + streaming speed | Mixed | Very high (avoid if possible) |

### 2.5 Data Partitioning
- Time-based partitioning for event data (partition by day/month)
- Hash partitioning for evenly distributed access patterns
- Partition pruning verified in query plans
- Partition maintenance automated (create future, archive old)

---

## 3. Data Quality Standards

### 3.1 Data Quality Dimensions

| Dimension | Definition | Measurement |
|-----------|-----------|-------------|
| Completeness | All required fields populated | % of non-null values in required columns |
| Accuracy | Data reflects real-world truth | Sampling + validation against source |
| Consistency | Same data produces same results | Cross-system reconciliation |
| Timeliness | Data available when needed | Pipeline SLA compliance |
| Uniqueness | No unintended duplicates | Duplicate count per key |
| Validity | Data conforms to defined rules | Constraint violation count |

### 3.2 Data Quality Checks (Mandatory)

Every pipeline must implement these checks:

**Schema checks:**
- Column count matches expectation
- Column types match expectation
- No unexpected null values in required columns

**Volume checks:**
- Row count within expected range (flag >50% deviation)
- File size within expected range
- No empty result sets (unless expected)

**Value checks:**
- Values within defined ranges (dates, amounts, percentages)
- Enum values match allowed set
- Referential integrity across datasets
- No duplicate primary keys

**Freshness checks:**
- Most recent timestamp within expected window
- Pipeline completion time within SLA

### 3.3 Data Quality Tools
- dbt tests for transformation validation
- Great Expectations or Soda for pipeline data quality
- Custom quality framework for domain-specific rules
- Quality metrics dashboard with historical trends
- Alerting on quality failures with severity classification

### 3.4 Data Quality SLAs

| Severity | Definition | Response |
|----------|-----------|----------|
| Critical | Data loss, corruption, or major inaccuracy | Fix within 2 hours, notify stakeholders |
| High | Significant quality degradation | Fix within 24 hours |
| Medium | Minor quality issue, workaround available | Fix within 1 week |
| Low | Cosmetic or edge case quality issue | Prioritize in backlog |

---

## 4. Data Governance Standards

### 4.1 Data Classification
- All data assets classified: Public, Internal, Confidential, Restricted
- Classification drives encryption, access control, and retention policies
- Classification is documented in data catalog metadata
- PII fields identified and tagged in all datasets

### 4.2 Data Catalog
- Every production dataset documented in the data catalog
- Catalog entries include: owner, description, schema, lineage, classification, SLA
- Data lineage tracked from source to consumption
- Catalog updated automatically from pipeline metadata
- Column-level descriptions for all business-critical fields

### 4.3 Data Access Control
- Access granted per dataset, not per database
- Read access requires business justification
- Write access restricted to owning pipelines
- PII access requires additional approval
- Access reviews conducted quarterly
- Access logs retained for audit compliance

### 4.4 Data Retention
- Retention policy defined per data classification
- Automated deletion/archival at retention expiry
- Retention exceptions documented and approved
- Deleted data verified as unrecoverable
- Legal hold capability for litigation preservation

---

## 5. Data Platform Standards

### 5.1 Data Lake / Lakehouse

**Storage layers:**
| Layer | Purpose | Format | Retention |
|-------|---------|--------|-----------|
| Raw / Bronze | Source data as-is | Parquet, JSON | Per retention policy |
| Cleaned / Silver | Validated, deduplicated, typed | Parquet (Delta/Iceberg) | Per retention policy |
| Curated / Gold | Business-ready, aggregated | Parquet (Delta/Iceberg) | Per retention policy |

**File format standards:**
- Parquet for columnar analytics (default)
- Delta Lake or Apache Iceberg for ACID transactions on data lake
- Avro for schema-evolution-heavy event data
- CSV only for data exchange with external systems (never internal)

### 5.2 Data Warehouse
- Separate schemas for: staging, core/marts, and reporting
- dbt or equivalent for transformation management
- All transformations version-controlled and tested
- Materialized views for frequently accessed aggregations
- Query performance monitoring and optimization

### 5.3 Real-Time Data
- Kafka or equivalent for event streaming
- Schema registry for event schema management
- Stream processing with Flink, Spark Streaming, or equivalent
- Exactly-once semantics where data accuracy requires it
- Backpressure handling and consumer lag monitoring

---

## 6. Testing Standards for Data

### 6.1 Unit Tests
- Test individual transformation functions
- Test data validation rules
- Test edge cases: nulls, empty strings, boundary values, special characters
- Test timezone handling
- Test numeric precision

### 6.2 Integration Tests
- Test full pipeline end-to-end with representative data
- Test idempotency: run pipeline twice, verify same result
- Test error handling: corrupt input, missing files, schema changes
- Test backfill: reprocess historical data, verify correctness

### 6.3 Data Reconciliation
- Cross-system reconciliation for critical data flows
- Source-to-target row count comparison
- Aggregate value comparison (sums, counts, averages)
- Reconciliation runs daily for critical pipelines

---

## 7. Operational Standards

### 7.1 Monitoring
- Pipeline execution status dashboard
- Data freshness monitoring per dataset
- Data quality score trends
- Resource utilization (compute, storage, network)
- Cost tracking per pipeline and per dataset

### 7.2 Alerting
| Condition | Severity | Response |
|-----------|----------|----------|
| Pipeline failure | High | Investigate, fix, rerun |
| SLA miss | High | Escalate to data team |
| Quality check failure | Medium-Critical | Depends on severity |
| Storage approaching limit | Warning | Capacity planning |
| Unusual data volume | Warning | Investigate source |

### 7.3 Documentation
- Pipeline documentation: purpose, schedule, dependencies, SLA
- Schema documentation: column descriptions, business meaning, valid values
- Runbook: common failures and resolution steps
- Data dictionary: business glossary mapped to technical fields

---

*Data is the foundation of every decision and every feature. Treat data infrastructure with the same rigor as application infrastructure. A system with unreliable data is a system that cannot be trusted.*
