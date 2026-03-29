# Domain Bridge: System Primitives Across Domains

> The kernel reasons in universal system primitives. Domain practitioners translate those primitives into the vocabulary of their field. This document shows how each primitive maps across multiple domains.

---

## The Translation Principle

The kernel's 11 system primitives are domain-agnostic by design. They describe the structural anatomy of any system — software, clinical, logistical, organizational, or physical. The same primitive means different things in different fields, but the structural role it plays is identical.

When the kernel produces a synthesis describing a system's "interfaces," a software engineer reads API contracts. A hospital administrator reads patient handoff protocols. A logistics planner reads carrier interchange agreements. A city planner reads road interchange specifications. The structural concept is the same. The domain vocabulary and implementation differ.

---

## Primitive Mapping Table

| Primitive | Software Systems | Healthcare Systems | Logistics Networks | Organizational Systems | Physical Infrastructure |
|-----------|-----------------|-------------------|-------------------|----------------------|------------------------|
| **Purpose** | Product requirements, user stories | Clinical objectives, patient outcomes | Delivery goals, service levels | Mission statement, strategic goals | Design specifications, service capacity |
| **Boundary** | Service boundaries, API contracts | Care pathway scope, department jurisdiction | Network coverage area, carrier handoffs | Organizational scope, team charters | Physical enclosure, right-of-way |
| **Inputs** | API requests, events, user actions | Patient intake, referral orders, lab results | Shipments, orders, vehicle arrivals | Requests, mandates, resources allocated | Load, force, environmental conditions |
| **Transformations** | Business logic, data processing | Clinical protocols, triage decisions | Route optimization, dispatch logic | Decision-making processes, approvals | Structural conversion, energy transfer |
| **Outputs** | API responses, rendered UI, events | Discharge summaries, treatment outcomes | Delivered goods, fulfillment records | Decisions, reports, services rendered | Structural product, processed output |
| **Interfaces** | REST APIs, gRPC, message queues | Patient handoffs, referral protocols | Carrier interchange, warehouse receipts | Meeting protocols, approval workflows | Load transfer points, connector specs |
| **Resources** | Compute, storage, network, cloud services | Staff, beds, equipment, medications | Vehicles, warehouses, drivers, fuel | Personnel, budget, tools, time | Materials, land, energy, equipment |
| **Constraints** | Latency SLAs, compliance (GDPR, HIPAA) | Regulatory requirements, staffing ratios | Weight limits, delivery windows, regulations | Budget ceilings, headcount limits, policy | Load limits, safety codes, material properties |
| **Feedback Loops** | Monitoring, alerting, analytics | Quality metrics, patient outcomes tracking | Delivery performance dashboards | KPI tracking, performance reviews | Structural monitoring, load sensors |
| **Failure Modes** | Service outages, data corruption, cascades | Adverse events, misdiagnosis, staffing gaps | Route failures, carrier defaults, delays | Decision deadlock, communication breakdown | Structural failure, overload, environmental damage |
| **Evolution Paths** | API versioning, migration strategies | Protocol updates, capacity expansion | Network expansion, carrier diversification | Reorganization, role evolution | Renovation, capacity upgrade, technology adoption |

---

## Using the Primitive Mapping

### For domain practitioners receiving kernel outputs

When the kernel synthesizes a system and defines its "interfaces," translate that to your domain's equivalent:
- Read "interface" as the connection points between components in your field
- The structural requirements (explicit contracts, failure behavior, versioning strategy) apply regardless of domain
- The specific format (API specification, handoff protocol document, carrier agreement) is your domain's convention

### For the kernel processing domain inputs

When domain practitioners provide domain-specific context, the kernel maps it back to primitives:
- A "staffing ratio" is a **constraint** on the **resources** primitive
- A "carrier default" is a **failure mode**
- A "protocol update" is an **evolution path**

This mapping ensures that domain knowledge is integrated into the structural model without contaminating the kernel's domain-agnostic reasoning.

---

## Domain-Specific Audit Evidence

The 8 universal backbone dimensions apply in every domain, but the evidence that supports a score is domain-specific.

| Dimension | Software Evidence | Healthcare Evidence | Logistics Evidence |
|-----------|-----------------|--------------------|--------------------|
| **Coherence** | All services trace to product requirements | All protocols trace to patient outcomes | All routes trace to delivery objectives |
| **Completeness** | All 11 primitives defined for every service | Every care step has defined protocols | Every network node has defined roles |
| **Failure Awareness** | Circuit breakers, DLQs, retry logic | Escalation protocols, triage criteria, backup staffing | Rerouting procedures, carrier redundancy, delay playbooks |
| **Adaptability** | API versioning, feature flags, modular boundaries | Protocol amendment processes, capacity flex plans | Network reconfiguration procedures, dynamic routing |
| **Implementability** | Team skills, tooling availability, timeline feasibility | Staffing capacity, equipment procurement, regulatory approval | Fleet availability, route permitting, partner agreements |

---

## Creating a New Domain Bridge

To apply the kernel to a new domain:

1. **Map all 11 primitives** to the domain's vocabulary. Every primitive must have a concrete, actionable translation — not an analogy, but an actual term practitioners use.

2. **Define domain-specific audit evidence**. For each of the 8 backbone dimensions, specify what a domain practitioner examines to assess quality in that domain.

3. **Identify characteristic failure modes**. Every domain has recurring failure patterns. Document them so the kernel's failure mode analysis is grounded in domain-specific risk.

4. **Define the deliverable format**. The canonical package is domain-neutral. Domain practitioners receive it and translate it into field-specific artifacts: engineering drawings, clinical pathway manuals, logistics operations guides, governance frameworks, construction plans.

5. **Register the bridge** in the kernel manifest (`kernel_manifest.yaml → domain_bridge`) so the Controller Architect can load the appropriate bridge based on the domain classification in the intent object.

---

*Domain Bridge — Universal Systems Kernel Integration Layer*
*The kernel does not change across domains. Only the bridge does.*
