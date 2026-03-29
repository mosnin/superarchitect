# Event-Driven Architecture Template

**Version:** 1.0
**Owner:** Architect Agent (Team 2)
**Last Updated:** 2026-03-29

---

## Purpose

This template provides the standard blueprint for event-driven systems. It covers event taxonomy, event design, broker selection, consumer patterns, error handling, and operational requirements. Use this template when the system requires loose coupling, real-time reactivity, or temporal decoupling between producers and consumers.

---

## 1. When to Use Event-Driven Architecture

**Strong signals for EDA:**
- Multiple consumers need to react to the same business event
- Producers should not know about or depend on consumers
- Temporal decoupling is needed (producer and consumer operate at different speeds)
- Real-time data processing or streaming is required
- System requires audit trails or event replay capability
- Integration with external systems via async messaging

**Signals against EDA:**
- Simple request-response patterns where caller needs immediate result
- Strong consistency requirements across operations
- Small team without operational maturity for async debugging
- Low throughput where the overhead of a message broker is not justified

---

## 2. Event Taxonomy

### Domain Events
Events that represent something meaningful that happened in the business domain.

**Naming:** Past tense verb describing what occurred.
**Examples:** `OrderPlaced`, `PaymentProcessed`, `InventoryReserved`, `UserRegistered`

**Rules:**
- Domain events are facts — they represent something that already happened
- They are immutable — once published, they cannot be changed
- They carry the minimum data needed for consumers to act
- They belong to the producing bounded context's ubiquitous language

### Integration Events
Events published specifically for cross-context communication, potentially with different schemas than internal domain events.

**Rules:**
- Integration events are the public contract — domain events are internal
- Schema evolution must be backward compatible
- Published through an anti-corruption layer that translates internal domain events

### System Events
Technical events for infrastructure concerns: deployment notifications, scaling events, health status changes, configuration updates.

---

## 3. Event Design Standards

### Event Envelope

Every event must include a standard envelope:

```json
{
  "eventId": "uuid-v4",
  "eventType": "com.company.context.EventName",
  "eventVersion": "1.0",
  "source": "service-name",
  "timestamp": "2026-03-29T10:15:30.123Z",
  "correlationId": "uuid-v4",
  "causationId": "uuid-v4",
  "traceId": "w3c-trace-id",
  "data": {
    // Event-specific payload
  },
  "metadata": {
    "userId": "usr-123",
    "tenantId": "tenant-456"
  }
}
```

### Event Payload Design

**Principle: Carry enough data for consumers to act without callbacks.**

- Include the entity ID and the data that changed
- For state-change events, include both old and new values when practical
- Do not include data the consumer does not need (minimizes coupling)
- Never include secrets, tokens, or PII beyond what is strictly necessary

### Event Naming Convention
```
<company>.<bounded-context>.<aggregate>.<event-name>.v<version>
Example: com.acme.orders.order.placed.v1
```

### Schema Evolution Rules

1. **Adding fields**: Always safe (new fields are optional with defaults)
2. **Removing fields**: Unsafe without versioning — consumers may depend on them
3. **Renaming fields**: Treat as remove old + add new (breaking)
4. **Changing field types**: Always breaking — requires new version

**Strategy:** Use a schema registry (Confluent, Apicurio, or AWS Glue). Validate schemas on publish. Support reading old versions (upcasting).

---

## 4. Message Broker Selection

| Broker | Best For | Throughput | Ordering | Retention |
|--------|----------|-----------|----------|-----------|
| Apache Kafka | High-throughput streaming, event sourcing | Millions/sec | Per partition | Configurable (days-forever) |
| RabbitMQ | Task queues, RPC, complex routing | Tens of thousands/sec | Per queue | Until consumed |
| AWS SQS/SNS | Serverless, simple pub/sub | Scalable | FIFO option | 14 days max |
| NATS JetStream | Low-latency, cloud-native | Millions/sec | Per stream | Configurable |
| Redis Streams | Simple streaming with existing Redis | Hundreds of thousands/sec | Per stream | Memory-bound |

### Selection Criteria
1. **Throughput requirements**: Messages per second at peak
2. **Ordering guarantees**: Global, per-key, or none
3. **Retention needs**: Must events be replayable? For how long?
4. **Delivery guarantee**: At-most-once, at-least-once, exactly-once
5. **Operational maturity**: Team's experience with the technology
6. **Cloud strategy**: Managed service availability

---

## 5. Producer Patterns

### Transactional Outbox (Recommended Default)

Guarantees atomicity between database write and event publish:

1. Within the same database transaction: write domain change + insert event into outbox table
2. Background relay process polls outbox and publishes to broker
3. Relay marks events as published after confirmed delivery
4. Consumers handle duplicates (at-least-once delivery)

### Change Data Capture (CDC)

Capture database changes from the transaction log and publish as events:

- Tools: Debezium (Kafka Connect), AWS DMS, Maxwell
- Best for: legacy integration, database-first workflows
- Limitation: events are row-level changes, not domain events (requires transformation)

### Direct Publish (Use with Caution)

Publish directly to the broker from application code:

- Risk: database write succeeds but publish fails (inconsistency)
- Acceptable only when: data loss is tolerable OR operations are idempotent
- Never use for: financial transactions, order processing, data mutations

---

## 6. Consumer Patterns

### Competing Consumers
Multiple instances of the same consumer process messages in parallel from a queue. Each message is delivered to exactly one consumer instance. Use for horizontal scaling of processing.

### Fan-Out
One event is delivered to multiple different consumer groups. Each group gets every message. Use when multiple bounded contexts react to the same event.

### Event Sourcing Consumer
Consumers that build read models (projections) from event streams. Must handle:
- Replay from the beginning (projection rebuild)
- Idempotent processing (same event processed twice produces same result)
- Checkpoint management (track position in the stream)

### Consumer Group Management
- Each logical consumer has a unique consumer group ID
- Consumer group tracks its offset/position in the stream
- Rebalancing occurs when consumers join/leave the group
- Monitor consumer lag (difference between latest event and consumer position)

---

## 7. Error Handling

### Dead Letter Queue (DLQ)

Every consumer must have a DLQ for messages that fail processing after retries:

1. Consumer attempts processing
2. On failure, retry with backoff (max 3-5 retries)
3. After max retries, move to DLQ
4. DLQ messages include: original message, error details, retry count, timestamp
5. Alerting on DLQ depth (non-zero DLQ requires investigation)
6. Manual or automated DLQ replay capability

### Poison Message Handling

Messages that can never be processed successfully (corrupt data, schema mismatch):

1. Detect: consecutive failures on the same message ID
2. Isolate: move to poison message queue immediately (no retries)
3. Alert: notify operations team
4. Analyze: determine root cause (schema change, data corruption, bug)
5. Resolve: fix consumer, replay from DLQ, or discard with documentation

### Backpressure

When consumers cannot keep up with producers:

1. **Buffer:** Broker retains messages (Kafka excels here)
2. **Throttle:** Rate-limit producers (only if producers are internal)
3. **Scale:** Add consumer instances (competing consumers pattern)
4. **Shed:** Drop low-priority messages (only with explicit business approval)
5. **Alert:** Consumer lag exceeding threshold triggers scaling or investigation

---

## 8. Exactly-Once Semantics

True exactly-once delivery is impossible in distributed systems. Achieve effectively-exactly-once through:

1. **Idempotent consumers**: Processing the same event twice produces the same result
2. **Idempotency key**: Store processed event IDs; skip duplicates
3. **Transactional consumer**: Commit message offset and database write in the same transaction (Kafka + relational DB)
4. **Deduplication window**: Track recent event IDs for a configurable window

---

## 9. Observability

### Mandatory Metrics
- **Producer**: Events published/sec, publish latency, publish errors
- **Broker**: Partition count, replication lag, storage usage
- **Consumer**: Events consumed/sec, processing latency, consumer lag, DLQ depth
- **End-to-end**: Event latency (publish timestamp to consumption timestamp)

### Tracing
- Inject trace context into event headers at publish time
- Extract and continue trace context at consume time
- Every event processing creates a child span linked to the original trace

### Alerting
| Condition | Severity | Action |
|-----------|----------|--------|
| Consumer lag > 10,000 events | Warning | Investigate, consider scaling |
| Consumer lag > 100,000 events | Critical | Scale consumers, investigate root cause |
| DLQ depth > 0 | Warning | Investigate failed messages |
| DLQ depth > 100 | Critical | Likely systematic failure, page on-call |
| Publish error rate > 1% | Critical | Broker health, network issues |

---

## 10. Testing Strategy

### Unit Tests
- Test event serialization/deserialization
- Test consumer logic with in-memory event fixtures
- Test idempotency (process same event twice, verify same result)

### Integration Tests
- Test publish and consume through real broker (testcontainers)
- Test DLQ behavior with intentionally failing messages
- Test consumer group rebalancing

### Contract Tests
- Verify event schema compatibility between producer and consumer
- Test schema evolution (old consumer reads new event format)

### Chaos Tests
- Broker node failure during publish
- Consumer crash during processing (verify no data loss)
- Network partition between producer and broker
- Slow consumer (verify backpressure handling)

---

## 11. Architecture Diagram Template

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Producer A     │────>│              │────>│   Consumer X     │
│  (Order Service) │     │              │     │  (Inventory Svc) │
└─────────────────┘     │   Message    │     └─────────────────┘
                        │   Broker     │
┌─────────────────┐     │              │     ┌─────────────────┐
│   Producer B     │────>│  (Kafka /    │────>│   Consumer Y     │
│  (Payment Svc)   │     │   RabbitMQ)  │     │  (Notification)  │
└─────────────────┘     │              │     └─────────────────┘
                        │              │
                        │              │     ┌─────────────────┐
                        │              │────>│   Consumer Z     │
                        └──────────────┘     │  (Analytics)     │
                               │             └─────────────────┘
                               │
                        ┌──────────────┐
                        │  Dead Letter  │
                        │    Queue      │
                        └──────────────┘
```

---

*Event-driven architecture trades immediate consistency for scalability and decoupling. This trade-off must be explicitly understood and accepted by all stakeholders before adoption.*
