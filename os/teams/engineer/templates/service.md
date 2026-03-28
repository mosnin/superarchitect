# Template: Background Service / Worker

A complete implementation template for a production-grade background service: job queue consumers, scheduled tasks, event processors, and long-running workers.

---

## Service Scaffold Structure

```
my-worker/
├── src/
│   ├── jobs/                          # One directory per job type
│   │   ├── <job-type>/
│   │   │   ├── <job-type>.processor.ts   # Core job processing logic
│   │   │   ├── <job-type>.types.ts       # Job payload type definitions
│   │   │   ├── <job-type>.handler.ts     # Queue binding: subscribe, ack, nack
│   │   │   ├── <job-type>.state.ts       # State machine (for long-running jobs)
│   │   │   └── <job-type>.processor.test.ts
│   │
│   ├── schedules/                     # Cron-style scheduled jobs
│   │   ├── <schedule-name>.task.ts
│   │   └── scheduler.ts              # Cron runner registration
│   │
│   ├── events/                        # Domain event consumers
│   │   ├── <event-name>.consumer.ts
│   │   └── event-bus.ts              # Event bus abstraction
│   │
│   ├── shared/
│   │   ├── config/
│   │   │   └── config.ts             # Parsed, validated config (same pattern as API)
│   │   ├── database/
│   │   │   └── database.ts           # Connection pool
│   │   ├── queue/
│   │   │   ├── queue.ts              # Queue client abstraction
│   │   │   ├── queue.impl.ts         # Concrete implementation (BullMQ, SQS, RabbitMQ)
│   │   │   └── dead-letter.ts        # DLQ handling
│   │   ├── errors/
│   │   │   ├── domain-error.ts
│   │   │   └── retry-policy.ts       # Retry/backoff configuration
│   │   ├── logging/
│   │   │   └── logger.ts
│   │   └── observability/
│   │       ├── metrics.ts            # Prometheus / StatsD counters, histograms
│   │       └── tracer.ts             # OpenTelemetry tracer
│   │
│   ├── health/
│   │   └── health-server.ts          # Minimal HTTP server for health probes
│   │
│   └── worker.ts                     # Entry point: wires all workers, starts consuming
│
├── test/
│   └── integration/
│
├── .env.example
├── Dockerfile
└── docker-compose.yml
```

---

## Job Queue Consumer Pattern

```typescript
// src/jobs/send-email/send-email.handler.ts
// Binds the processor to the queue infrastructure

import { Queue, Job } from '../shared/queue/queue'
import { SendEmailProcessor } from './send-email.processor'
import { SendEmailPayload } from './send-email.types'
import { Logger } from '../shared/logging/logger'
import { metrics } from '../shared/observability/metrics'

export class SendEmailHandler {
  constructor(
    private readonly queue: Queue,
    private readonly processor: SendEmailProcessor,
    private readonly logger: Logger,
  ) {}

  start(): void {
    this.queue.consume<SendEmailPayload>('send-email', async (job: Job<SendEmailPayload>) => {
      const span = tracer.startSpan('job.send-email', {
        attributes: { 'job.id': job.id, 'job.attempt': job.attemptsMade },
      })

      try {
        this.logger.info('Processing job', {
          jobId: job.id,
          jobType: 'send-email',
          attempt: job.attemptsMade,
          payload: redactSensitive(job.data),
        })

        await this.processor.process(job.data)

        metrics.jobCompleted.inc({ job_type: 'send-email', status: 'success' })
        this.logger.info('Job completed', { jobId: job.id, jobType: 'send-email' })
        span.setStatus({ code: SpanStatusCode.OK })
      } catch (err) {
        metrics.jobCompleted.inc({ job_type: 'send-email', status: 'error' })
        this.logger.error('Job failed', {
          jobId: job.id,
          jobType: 'send-email',
          attempt: job.attemptsMade,
          error: err,
        })
        span.setStatus({ code: SpanStatusCode.ERROR, message: (err as Error).message })
        throw err  // Re-throw so the queue client handles retry/DLQ
      } finally {
        span.end()
      }
    })
  }
}
```

**Queue abstraction:**
```typescript
// src/shared/queue/queue.ts
// Abstract interface: lets you swap BullMQ for SQS without changing consumers

export interface Job<T> {
  id: string
  data: T
  attemptsMade: number
}

export interface Queue {
  consume<T>(queueName: string, processor: (job: Job<T>) => Promise<void>): void
  enqueue<T>(queueName: string, payload: T, options?: EnqueueOptions): Promise<string>
  close(): Promise<void>
}

export interface EnqueueOptions {
  delay?: number       // Delay in milliseconds
  priority?: number    // Higher = processed first
  attempts?: number    // Override default retry count
  jobId?: string       // Idempotency key
}
```

---

## Scheduled Job Pattern

```typescript
// src/schedules/cleanup-expired-tokens.task.ts
// Runs on a cron schedule to purge soft-deleted records

import { CronJob } from 'cron'
import { Knex } from 'knex'
import { Logger } from '../shared/logging/logger'
import { metrics } from '../shared/observability/metrics'

export class CleanupExpiredTokensTask {
  private job: CronJob

  constructor(
    private readonly db: Knex,
    private readonly logger: Logger,
    // Schedule: every day at 2am UTC
    private readonly schedule: string = '0 2 * * *',
  ) {
    this.job = new CronJob(this.schedule, () => this.run(), null, false, 'UTC')
  }

  start(): void {
    this.job.start()
    this.logger.info('Scheduled task registered', {
      task: 'cleanup-expired-tokens',
      schedule: this.schedule,
      nextRun: this.job.nextDate().toISO(),
    })
  }

  stop(): void {
    this.job.stop()
    this.logger.info('Scheduled task stopped', { task: 'cleanup-expired-tokens' })
  }

  async run(): Promise<void> {
    const timer = metrics.taskDuration.startTimer({ task: 'cleanup-expired-tokens' })
    this.logger.info('Starting scheduled task', { task: 'cleanup-expired-tokens' })

    try {
      const deletedCount = await this.db('refresh_tokens')
        .where('expires_at', '<', new Date())
        .delete()

      metrics.taskCompleted.inc({ task: 'cleanup-expired-tokens', status: 'success' })
      this.logger.info('Scheduled task complete', {
        task: 'cleanup-expired-tokens',
        deletedCount,
        durationMs: timer(),
      })
    } catch (err) {
      metrics.taskCompleted.inc({ task: 'cleanup-expired-tokens', status: 'error' })
      this.logger.error('Scheduled task failed', {
        task: 'cleanup-expired-tokens',
        error: err,
        durationMs: timer(),
      })
    }
  }
}
```

---

## Event Consumer Pattern

```typescript
// src/events/user-created.consumer.ts
// Reacts to domain events published by other services

import { EventBus, DomainEvent } from '../shared/queue/event-bus'
import { UserCreatedPayload } from './user-created.types'
import { WelcomeEmailService } from '../shared/email/welcome-email.service'

export class UserCreatedConsumer {
  constructor(
    private readonly eventBus: EventBus,
    private readonly welcomeEmailService: WelcomeEmailService,
    private readonly logger: Logger,
  ) {}

  register(): void {
    this.eventBus.subscribe<UserCreatedPayload>('user.created', async (event: DomainEvent<UserCreatedPayload>) => {
      this.logger.info('Handling event', {
        eventType: 'user.created',
        eventId: event.id,
        userId: event.payload.userId,
      })

      // Idempotency check: have we already processed this event?
      const alreadyProcessed = await this.eventBus.isProcessed(event.id)
      if (alreadyProcessed) {
        this.logger.info('Skipping duplicate event', { eventId: event.id })
        return
      }

      await this.welcomeEmailService.sendWelcome({
        userId: event.payload.userId,
        email: event.payload.email,
        name: event.payload.name,
      })

      await this.eventBus.markProcessed(event.id)
    })
  }
}
```

---

## Retry/Backoff Pattern Implementation

```typescript
// src/shared/errors/retry-policy.ts

export interface RetryPolicy {
  maxAttempts: number
  initialDelayMs: number
  maxDelayMs: number
  backoffMultiplier: number
  jitterFactor: number       // 0–1: adds randomness to avoid thundering herd
  retryableErrors: string[]  // Error codes that warrant a retry (vs. fatal errors)
}

export const defaultRetryPolicy: RetryPolicy = {
  maxAttempts: 5,
  initialDelayMs: 1_000,      // 1 second
  maxDelayMs: 300_000,         // 5 minutes
  backoffMultiplier: 2,        // Exponential: 1s, 2s, 4s, 8s, 16s...
  jitterFactor: 0.25,          // Up to ±25% randomness on each delay
  retryableErrors: [
    'NETWORK_TIMEOUT',
    'UPSTREAM_UNAVAILABLE',
    'RATE_LIMITED',
    'DB_CONNECTION_LOST',
  ],
}

export function calculateDelay(attempt: number, policy: RetryPolicy): number {
  const baseDelay = policy.initialDelayMs * Math.pow(policy.backoffMultiplier, attempt - 1)
  const cappedDelay = Math.min(baseDelay, policy.maxDelayMs)
  const jitter = cappedDelay * policy.jitterFactor * (Math.random() * 2 - 1)  // ±jitterFactor
  return Math.max(0, cappedDelay + jitter)
}

export function isRetryable(error: unknown, policy: RetryPolicy): boolean {
  if (error instanceof DomainError) {
    return policy.retryableErrors.includes(error.code)
  }
  // Unknown errors (network, runtime) are retried by default
  return true
}

// Queue-level retry configuration (BullMQ example)
export function toBullMQRetryOptions(policy: RetryPolicy) {
  return {
    attempts: policy.maxAttempts,
    backoff: {
      type: 'exponential',
      delay: policy.initialDelayMs,
    },
    removeOnComplete: { count: 100 },
    removeOnFail: false,  // Keep failed jobs for inspection
  }
}
```

**Manual retry with backoff (for code outside the queue framework):**
```typescript
async function withRetry<T>(
  operation: () => Promise<T>,
  policy: RetryPolicy,
  logger: Logger,
): Promise<T> {
  let lastError: unknown

  for (let attempt = 1; attempt <= policy.maxAttempts; attempt++) {
    try {
      return await operation()
    } catch (err) {
      lastError = err

      if (attempt === policy.maxAttempts || !isRetryable(err, policy)) {
        throw err
      }

      const delay = calculateDelay(attempt, policy)
      logger.warn('Operation failed, retrying', {
        attempt,
        maxAttempts: policy.maxAttempts,
        delayMs: delay,
        error: (err as Error).message,
      })

      await sleep(delay)
    }
  }

  throw lastError
}
```

---

## Dead Letter Handling

```typescript
// src/shared/queue/dead-letter.ts
// Handle jobs that exhausted all retries

export class DeadLetterHandler {
  constructor(
    private readonly db: Knex,
    private readonly alertingService: AlertingService,
    private readonly logger: Logger,
  ) {}

  async handle(job: FailedJob): Promise<void> {
    // 1. Persist the failed job for forensic investigation
    await this.db('dead_letter_queue').insert({
      id: randomUUID(),
      original_job_id: job.id,
      job_type: job.name,
      payload: JSON.stringify(job.data),
      last_error: job.failedReason,
      attempts_made: job.attemptsMade,
      first_failed_at: job.processedOn ? new Date(job.processedOn) : null,
      created_at: new Date(),
    })

    // 2. Alert on-call if this job type is critical
    if (this.isCriticalJobType(job.name)) {
      await this.alertingService.page({
        title: `Critical job failed: ${job.name}`,
        message: job.failedReason,
        context: { jobId: job.id, jobType: job.name },
        severity: 'high',
      })
    }

    this.logger.error('Job moved to dead letter queue', {
      jobId: job.id,
      jobType: job.name,
      attempts: job.attemptsMade,
      error: job.failedReason,
    })
  }

  // Manual replay: re-enqueue a dead letter job after the underlying issue is fixed
  async replay(deadLetterId: string, queue: Queue): Promise<void> {
    const record = await this.db('dead_letter_queue').where({ id: deadLetterId }).first()
    if (!record) throw new Error(`Dead letter record ${deadLetterId} not found`)

    const newJobId = await queue.enqueue(record.job_type, JSON.parse(record.payload))

    await this.db('dead_letter_queue')
      .where({ id: deadLetterId })
      .update({ replayed_at: new Date(), replayed_job_id: newJobId })

    this.logger.info('Dead letter job replayed', {
      deadLetterId,
      originalJobId: record.original_job_id,
      newJobId,
    })
  }

  private isCriticalJobType(jobType: string): boolean {
    const criticalTypes = ['process-payment', 'send-verification-email', 'fraud-check']
    return criticalTypes.includes(jobType)
  }
}
```

---

## State Machine Pattern for Long-Running Jobs

Use a state machine when a job can be in one of several states, transitions between states are meaningful, and the job may take minutes or hours.

```typescript
// src/jobs/export-report/export-report.state.ts

export enum ExportReportState {
  PENDING = 'pending',
  FETCHING_DATA = 'fetching_data',
  TRANSFORMING = 'transforming',
  GENERATING_FILE = 'generating_file',
  UPLOADING = 'uploading',
  NOTIFYING = 'notifying',
  COMPLETED = 'completed',
  FAILED = 'failed',
}

// Valid state transitions — if a transition is not in this map, it is rejected
const VALID_TRANSITIONS: Record<ExportReportState, ExportReportState[]> = {
  [ExportReportState.PENDING]: [ExportReportState.FETCHING_DATA, ExportReportState.FAILED],
  [ExportReportState.FETCHING_DATA]: [ExportReportState.TRANSFORMING, ExportReportState.FAILED],
  [ExportReportState.TRANSFORMING]: [ExportReportState.GENERATING_FILE, ExportReportState.FAILED],
  [ExportReportState.GENERATING_FILE]: [ExportReportState.UPLOADING, ExportReportState.FAILED],
  [ExportReportState.UPLOADING]: [ExportReportState.NOTIFYING, ExportReportState.FAILED],
  [ExportReportState.NOTIFYING]: [ExportReportState.COMPLETED, ExportReportState.FAILED],
  [ExportReportState.COMPLETED]: [],
  [ExportReportState.FAILED]: [ExportReportState.PENDING],  // Allow retry/reset
}

export class ExportReportJob {
  private state: ExportReportState
  private readonly jobId: string

  constructor(jobId: string, initialState = ExportReportState.PENDING) {
    this.jobId = jobId
    this.state = initialState
  }

  transition(to: ExportReportState, db: Knex, logger: Logger): void {
    const allowed = VALID_TRANSITIONS[this.state]
    if (!allowed.includes(to)) {
      throw new Error(`Invalid transition: ${this.state} → ${to} for job ${this.jobId}`)
    }

    logger.info('Job state transition', {
      jobId: this.jobId,
      from: this.state,
      to,
    })

    this.state = to
    // Persist state change (for resumability on crash)
    db('export_jobs').where({ id: this.jobId }).update({
      state: to,
      updated_at: new Date(),
    })
  }

  getState(): ExportReportState {
    return this.state
  }
}

// Processor using the state machine
export class ExportReportProcessor {
  async process(job: ExportReportJob, payload: ExportReportPayload): Promise<void> {
    job.transition(ExportReportState.FETCHING_DATA, this.db, this.logger)
    const rawData = await this.dataService.fetch(payload.reportId)

    job.transition(ExportReportState.TRANSFORMING, this.db, this.logger)
    const transformed = await this.transformer.transform(rawData, payload.format)

    job.transition(ExportReportState.GENERATING_FILE, this.db, this.logger)
    const fileBuffer = await this.fileGenerator.generate(transformed)

    job.transition(ExportReportState.UPLOADING, this.db, this.logger)
    const fileUrl = await this.storageService.upload(fileBuffer, `reports/${payload.reportId}.${payload.format}`)

    job.transition(ExportReportState.NOTIFYING, this.db, this.logger)
    await this.notificationService.notify(payload.userId, { reportUrl: fileUrl })

    job.transition(ExportReportState.COMPLETED, this.db, this.logger)
  }
}
```

---

## Observability Integration

```typescript
// src/shared/observability/metrics.ts
// Prometheus-compatible metrics

import { Counter, Histogram, Gauge, Registry } from 'prom-client'

const registry = new Registry()

export const metrics = {
  // Job processing counters
  jobCompleted: new Counter({
    name: 'worker_jobs_total',
    help: 'Total number of jobs processed',
    labelNames: ['job_type', 'status'],  // status: success | error
    registers: [registry],
  }),

  // Job duration histogram (for latency percentiles)
  jobDuration: new Histogram({
    name: 'worker_job_duration_seconds',
    help: 'Duration of job processing in seconds',
    labelNames: ['job_type'],
    buckets: [0.1, 0.5, 1, 2, 5, 10, 30, 60],
    registers: [registry],
  }),

  // Queue depth (how many jobs are waiting)
  queueDepth: new Gauge({
    name: 'worker_queue_depth',
    help: 'Number of jobs currently in the queue',
    labelNames: ['queue_name'],
    registers: [registry],
  }),

  // Scheduled task counters
  taskCompleted: new Counter({
    name: 'worker_scheduled_tasks_total',
    help: 'Total scheduled task executions',
    labelNames: ['task', 'status'],
    registers: [registry],
  }),

  taskDuration: new Histogram({
    name: 'worker_scheduled_task_duration_seconds',
    help: 'Duration of scheduled task execution',
    labelNames: ['task'],
    buckets: [0.1, 1, 5, 30, 60, 300],
    registers: [registry],
  }),
}

export { registry }
```

```typescript
// src/shared/observability/tracer.ts
// OpenTelemetry distributed tracing

import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'
import { Resource } from '@opentelemetry/resources'
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions'

export function initTracing(config: TracingConfig): void {
  const provider = new NodeTracerProvider({
    resource: new Resource({
      [SemanticResourceAttributes.SERVICE_NAME]: config.serviceName,
      [SemanticResourceAttributes.SERVICE_VERSION]: config.serviceVersion,
    }),
  })

  provider.addSpanProcessor(
    new BatchSpanProcessor(
      new OTLPTraceExporter({ url: config.exporterEndpoint })
    )
  )

  provider.register()
}

export const tracer = trace.getTracer('worker')
```

---

## Health Server for Worker Processes

```typescript
// src/health/health-server.ts
// Workers don't serve HTTP, but Kubernetes still needs health probes.
// Expose a minimal HTTP server on a separate port just for probes.

import http from 'http'
import { Queue } from '../shared/queue/queue'
import { Knex } from 'knex'

export function startHealthServer(port: number, db: Knex, queue: Queue): http.Server {
  const server = http.createServer(async (req, res) => {
    if (req.url === '/health/live') {
      res.writeHead(200, { 'Content-Type': 'application/json' })
      res.end(JSON.stringify({ status: 'ok' }))
      return
    }

    if (req.url === '/health/ready') {
      const checks: Record<string, string> = {}
      let ready = true

      try {
        await db.raw('SELECT 1')
        checks.database = 'ok'
      } catch {
        checks.database = 'error'
        ready = false
      }

      try {
        await queue.ping()
        checks.queue = 'ok'
      } catch {
        checks.queue = 'error'
        ready = false
      }

      res.writeHead(ready ? 200 : 503, { 'Content-Type': 'application/json' })
      res.end(JSON.stringify({ status: ready ? 'ready' : 'degraded', checks }))
      return
    }

    if (req.url === '/metrics') {
      const { registry } = await import('../shared/observability/metrics')
      res.writeHead(200, { 'Content-Type': registry.contentType })
      res.end(await registry.metrics())
      return
    }

    res.writeHead(404)
    res.end()
  })

  server.listen(port)
  return server
}
```

---

## Graceful Shutdown with Job Drain

```typescript
// src/worker.ts
// Entry point for the worker process

async function main() {
  initTracing(config.tracing)
  const logger = createLogger({ service: config.serviceName, version: config.version })
  const db = await createDatabase(config.database)
  const queue = createQueue(config.queue)

  // Wire up all job handlers
  const sendEmailHandler = new SendEmailHandler(queue, new SendEmailProcessor(...), logger)
  const exportReportHandler = new ExportReportHandler(queue, new ExportReportProcessor(...), logger)

  // Wire up all event consumers
  const userCreatedConsumer = new UserCreatedConsumer(eventBus, welcomeEmailService, logger)

  // Wire up all scheduled tasks
  const cleanupTask = new CleanupExpiredTokensTask(db, logger)

  // Start consuming
  sendEmailHandler.start()
  exportReportHandler.start()
  userCreatedConsumer.register()
  cleanupTask.start()

  // Health probe server
  const healthServer = startHealthServer(config.healthPort, db, queue)

  logger.info('Worker started', {
    jobs: ['send-email', 'export-report'],
    events: ['user.created'],
    schedules: ['cleanup-expired-tokens'],
  })

  // Graceful shutdown
  const shutdown = async (signal: string) => {
    logger.info(`Received ${signal}, draining jobs...`)

    // 1. Stop accepting new jobs from the queue
    await queue.pause()

    // 2. Wait for in-flight jobs to complete (with timeout)
    const drainTimeout = setTimeout(() => {
      logger.error('Job drain timed out, forcing shutdown')
      process.exit(1)
    }, config.shutdownTimeoutMs)

    await queue.drain()
    clearTimeout(drainTimeout)
    logger.info('All in-flight jobs completed')

    // 3. Stop scheduled tasks (no new runs)
    cleanupTask.stop()

    // 4. Close connections
    await queue.close()
    await db.destroy()
    healthServer.close()

    logger.info('Worker shutdown complete')
    process.exit(0)
  }

  process.on('SIGTERM', () => shutdown('SIGTERM'))
  process.on('SIGINT', () => shutdown('SIGINT'))
}

main().catch((err) => {
  console.error('Failed to start worker:', err)
  process.exit(1)
})
```

**Shutdown sequence:**
```
1. SIGTERM received
2. queue.pause() — no new jobs dequeued
3. Wait for active jobs to complete (max SHUTDOWN_TIMEOUT_MS, default 30s)
4. Stop scheduled task cron runners
5. queue.close() — close connection to broker
6. db.destroy() — close database connection pool
7. healthServer.close() — stop health probe server
8. process.exit(0)

If drain timeout fires: log error, process.exit(1)
Kubernetes restarts the pod; any incomplete jobs are re-queued on startup.
```

---

## Complete Example: Email Notification Service

Processes email send requests from a queue. Supports multiple email types (welcome, password reset, order confirmation). Implements retry with backoff, dead letter handling, and observability.

**Directory:**
```
src/
  jobs/
    send-email/
      send-email.processor.ts     # Core logic: pick template, render, send via provider
      send-email.handler.ts       # Queue binding
      send-email.types.ts         # SendEmailPayload shape
      templates/
        welcome.template.ts       # Welcome email template renderer
        password-reset.template.ts
        order-confirmation.template.ts
  shared/
    email/
      email-provider.ts           # Interface for SendGrid, SES, Mailgun
      sendgrid.provider.ts        # SendGrid implementation
      email-template.ts           # Template engine wrapper
```

**SendEmailPayload:**
```typescript
type EmailType = 'welcome' | 'password-reset' | 'order-confirmation'

interface SendEmailPayload {
  idempotencyKey: string          // Prevents duplicate sends on retry
  to: string                      // Recipient email
  emailType: EmailType
  data: Record<string, unknown>   // Template-specific data
  metadata: {
    userId?: string
    orderId?: string
    requestedAt: string           // ISO 8601: when the send was originally requested
  }
}
```

**Processor logic:**
```
1. Check idempotency: look up idempotencyKey in `email_sends` table
   - If SENT: log "skipping duplicate", return success (idempotent)
   - If PENDING or not found: continue

2. Record attempt in `email_sends` table (upsert: idempotencyKey, status=PENDING)

3. Select template renderer based on emailType

4. Render HTML and text content from template + data
   - Validate that required template variables are present
   - Sanitize any user-provided data to prevent HTML injection

5. Send via email provider (SendGrid, SES, etc.)
   - Include headers: X-Entity-Ref-ID (idempotencyKey), X-Mailer (service name + version)

6. Update `email_sends` table: status=SENT, sent_at=now(), provider_message_id

7. Log success with duration metric

On failure:
  - If provider returns 4xx (invalid recipient, rejected): mark PERMANENTLY_FAILED, do NOT retry
  - If provider returns 5xx or network timeout: throw error (queue retries with backoff)
  - If max retries exhausted: dead letter handler persists record, alerts on-call if emailType is critical
```

**Retry policy:**
```typescript
const emailRetryPolicy: RetryPolicy = {
  maxAttempts: 5,
  initialDelayMs: 2_000,      // 2 seconds
  maxDelayMs: 600_000,         // 10 minutes
  backoffMultiplier: 3,        // 2s, 6s, 18s, 54s, 162s (~2.7 min)
  jitterFactor: 0.2,
  retryableErrors: ['EMAIL_PROVIDER_UNAVAILABLE', 'RATE_LIMITED', 'NETWORK_TIMEOUT'],
}
```

**Database table:**
```sql
CREATE TABLE email_sends (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  idempotency_key VARCHAR(255) NOT NULL UNIQUE,
  email_type VARCHAR(50) NOT NULL,
  to_address VARCHAR(255) NOT NULL,
  status VARCHAR(50) NOT NULL DEFAULT 'pending',  -- pending, sent, permanently_failed
  provider_message_id VARCHAR(255),
  attempts_made INTEGER NOT NULL DEFAULT 0,
  last_error TEXT,
  user_id UUID,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  sent_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX email_sends_idempotency_key_idx ON email_sends(idempotency_key);
CREATE INDEX email_sends_user_id_idx ON email_sends(user_id);
CREATE INDEX email_sends_status_created_at_idx ON email_sends(status, created_at);
```

**Observability:**
```
Metrics emitted:
  worker_jobs_total{job_type="send-email", status="success"} — counter
  worker_jobs_total{job_type="send-email", status="error"} — counter
  worker_job_duration_seconds{job_type="send-email"} — histogram
  email_sends_total{email_type="welcome", status="sent"} — counter (business metric)
  email_provider_latency_seconds{provider="sendgrid"} — histogram

Logs emitted (structured JSON):
  INFO  "Processing job"       {jobId, emailType, to (masked), attempt}
  INFO  "Skipping duplicate"   {jobId, idempotencyKey}
  INFO  "Email sent"           {jobId, messageId, durationMs}
  WARN  "Email send retry"     {jobId, attempt, maxAttempts, delayMs, error}
  ERROR "Email permanently failed" {jobId, emailType, reason}
  ERROR "Job moved to DLQ"     {jobId, attempts, error}

Traces:
  Span: job.send-email
    Child span: template.render
    Child span: email-provider.send
```
