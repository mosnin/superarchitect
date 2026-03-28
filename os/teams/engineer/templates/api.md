# Template: Production REST API Service

A complete implementation template for a production-grade REST API service. Follow this template for any HTTP API that serves external or internal consumers.

---

## Service Scaffold Structure

```
my-api/
├── src/
│   ├── <feature>/                    # One directory per feature/domain
│   │   ├── <feature>.model.ts        # Domain model / entity
│   │   ├── <feature>.types.ts        # TypeScript types/interfaces for the feature
│   │   ├── <feature>.repository.ts   # Repository interface (contract)
│   │   ├── <feature>.repository.impl.ts  # Concrete repository implementation
│   │   ├── <feature>.service.ts      # Business logic
│   │   ├── <feature>.handler.ts      # HTTP route handler
│   │   ├── <feature>.validator.ts    # Input validation schemas
│   │   ├── <feature>.service.test.ts # Unit tests for service
│   │   ├── <feature>.repository.integration.test.ts
│   │   ├── <feature>.handler.integration.test.ts
│   │   └── index.ts                  # Public exports from this feature
│   │
│   ├── shared/
│   │   ├── config/
│   │   │   ├── config.ts             # Parsed, validated config object
│   │   │   └── config.schema.ts      # Zod/Joi/class-validator schema
│   │   ├── database/
│   │   │   ├── database.ts           # Connection factory and pool management
│   │   │   ├── migrations/           # Numbered migration files
│   │   │   └── seeds/                # Test/dev seed data
│   │   ├── errors/
│   │   │   ├── domain-error.ts       # Typed domain error class
│   │   │   └── error-codes.ts        # Enum of all error codes
│   │   ├── logging/
│   │   │   └── logger.ts             # Logger factory (structured, leveled)
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts     # JWT verification
│   │   │   ├── rate-limit.middleware.ts
│   │   │   ├── request-id.middleware.ts  # Attach correlation ID
│   │   │   ├── error-handler.middleware.ts  # Global error handler
│   │   │   ├── logging.middleware.ts  # Request/response logging
│   │   │   └── cors.middleware.ts
│   │   └── http/
│   │       ├── response.ts           # Typed response helpers (ok, created, noContent)
│   │       └── router.ts             # Base router factory
│   │
│   ├── health/
│   │   └── health.handler.ts         # /health and /ready endpoints
│   │
│   ├── app.ts                        # Express app factory (testable, no listen)
│   └── server.ts                     # Entry point: wires app, calls listen, handles signals
│
├── test/
│   └── e2e/                          # End-to-end tests against running server
│
├── openapi.yaml                      # API specification
├── .env.example                      # All environment variable names with descriptions
├── Dockerfile
├── docker-compose.yml                # Local dev with DB, Redis, etc.
└── package.json
```

### Every file's responsibility:

| File | Responsibility |
|---|---|
| `server.ts` | Calls `app.listen()`, registers signal handlers, orchestrates graceful shutdown |
| `app.ts` | Creates and configures the Express app, mounts middleware and routes. Exported for testing. |
| `*.handler.ts` | Parses HTTP request, validates input, calls service, maps result to HTTP response |
| `*.service.ts` | Business logic only. No HTTP concepts, no database drivers. Depends on repository interfaces. |
| `*.repository.ts` | Interface (abstract contract) defining the data access methods the service needs |
| `*.repository.impl.ts` | Concrete implementation of the repository interface using a specific database/ORM |
| `*.model.ts` | Domain entity: pure data structure + domain methods. No I/O, no HTTP. |
| `*.validator.ts` | Input validation schema definitions (Zod, Joi, class-validator) |
| `config.ts` | Reads environment, validates all values at startup, exports typed config object |
| `database.ts` | Creates and manages the database connection pool |
| `domain-error.ts` | Error class carrying code, message, HTTP status, and optional context |
| `logger.ts` | Structured logger factory; injects service name, version, request ID |
| `health.handler.ts` | `/health` (liveness) and `/ready` (readiness) endpoints |

---

## Main Entry Point Pattern

```typescript
// src/server.ts
// Entry point: wire dependencies, start server, handle shutdown

import { createApp } from './app'
import { config } from './shared/config/config'
import { createDatabase } from './shared/database/database'
import { createLogger } from './shared/logging/logger'
import { UserRepositoryImpl } from './users/user.repository.impl'
import { UserService } from './users/user.service'

const logger = createLogger({ service: config.serviceName, version: config.version })

async function main() {
  // 1. Initialize infrastructure
  const db = await createDatabase(config.database)
  logger.info('Database connected')

  // 2. Wire dependencies (manual DI — explicit, no magic)
  const userRepository = new UserRepositoryImpl(db)
  const userService = new UserService(userRepository)

  // 3. Create app with all wired dependencies
  const app = createApp({ userService, config, logger })

  // 4. Start listening
  const server = app.listen(config.port, () => {
    logger.info(`Server listening`, { port: config.port, env: config.env })
  })

  // 5. Graceful shutdown
  const shutdown = async (signal: string) => {
    logger.info(`Received ${signal}, starting graceful shutdown`)

    server.close(async () => {
      logger.info('HTTP server closed')
      await db.destroy()
      logger.info('Database connections closed')
      process.exit(0)
    })

    // Force shutdown if graceful close takes too long
    setTimeout(() => {
      logger.error('Forced shutdown after timeout')
      process.exit(1)
    }, config.shutdownTimeoutMs)
  }

  process.on('SIGTERM', () => shutdown('SIGTERM'))
  process.on('SIGINT', () => shutdown('SIGINT'))
}

main().catch((err) => {
  logger.fatal('Failed to start server', { error: err })
  process.exit(1)
})
```

---

## Router/Handler Pattern

```typescript
// src/users/user.handler.ts
// HTTP layer: parse input, validate, call service, return response

import { Router, Request, Response, NextFunction } from 'express'
import { UserService } from './user.service'
import { createUserSchema, updateUserSchema } from './user.validator'
import { DomainError } from '../shared/errors/domain-error'

export function createUserRouter(userService: UserService): Router {
  const router = Router()

  // GET /users — list with pagination
  router.get('/', async (req: Request, res: Response, next: NextFunction) => {
    try {
      const { cursor, limit = 20 } = req.query
      const result = await userService.listUsers({
        cursor: cursor as string | undefined,
        limit: Math.min(Number(limit), 100),
      })
      res.status(200).json(result)
    } catch (err) {
      next(err)
    }
  })

  // GET /users/:id
  router.get('/:id', async (req: Request, res: Response, next: NextFunction) => {
    try {
      const user = await userService.getUserById(req.params.id)
      res.status(200).json({ data: user })
    } catch (err) {
      next(err)
    }
  })

  // POST /users
  router.post('/', async (req: Request, res: Response, next: NextFunction) => {
    try {
      const parsed = createUserSchema.safeParse(req.body)
      if (!parsed.success) {
        return res.status(422).json(formatValidationError(parsed.error))
      }
      const user = await userService.createUser(parsed.data)
      res.status(201).location(`/users/${user.id}`).json({ data: user })
    } catch (err) {
      next(err)
    }
  })

  // PATCH /users/:id
  router.patch('/:id', async (req: Request, res: Response, next: NextFunction) => {
    try {
      const parsed = updateUserSchema.safeParse(req.body)
      if (!parsed.success) {
        return res.status(422).json(formatValidationError(parsed.error))
      }
      const user = await userService.updateUser(req.params.id, parsed.data)
      res.status(200).json({ data: user })
    } catch (err) {
      next(err)
    }
  })

  // DELETE /users/:id
  router.delete('/:id', async (req: Request, res: Response, next: NextFunction) => {
    try {
      await userService.deleteUser(req.params.id)
      res.status(204).send()
    } catch (err) {
      next(err)
    }
  })

  return router
}
```

---

## Service Layer Pattern

```typescript
// src/users/user.service.ts
// Business logic: no HTTP concepts, no database drivers

import { UserRepository } from './user.repository'
import { User } from './user.model'
import { CreateUserInput, UpdateUserInput } from './user.types'
import { DomainError, ErrorCode } from '../shared/errors/domain-error'
import { hashPassword, verifyPassword } from '../shared/crypto/password'

export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async getUserById(id: string): Promise<User> {
    const user = await this.userRepository.findById(id)
    if (!user) {
      throw new DomainError({
        code: ErrorCode.USER_NOT_FOUND,
        message: `User ${id} does not exist`,
        httpStatus: 404,
        context: { userId: id },
      })
    }
    return user
  }

  async createUser(input: CreateUserInput): Promise<User> {
    const existing = await this.userRepository.findByEmail(input.email)
    if (existing) {
      throw new DomainError({
        code: ErrorCode.DUPLICATE_USER,
        message: 'A user with this email already exists',
        httpStatus: 409,
        context: { email: input.email },
      })
    }

    const passwordHash = await hashPassword(input.password)
    const user = User.create({ ...input, passwordHash })
    return this.userRepository.save(user)
  }

  async updateUser(id: string, input: UpdateUserInput): Promise<User> {
    const user = await this.getUserById(id)  // throws 404 if not found
    user.update(input)
    return this.userRepository.save(user)
  }

  async deleteUser(id: string): Promise<void> {
    const user = await this.getUserById(id)  // throws 404 if not found
    user.softDelete()
    await this.userRepository.save(user)
  }

  async listUsers(params: { cursor?: string; limit: number }) {
    return this.userRepository.findMany(params)
  }
}
```

---

## Repository Layer Pattern

```typescript
// src/users/user.repository.ts — Interface (contract)
import { User } from './user.model'

export interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  findMany(params: { cursor?: string; limit: number }): Promise<{
    data: User[]
    pagination: { cursor: string | null; hasMore: boolean }
  }>
  save(user: User): Promise<User>
}

// src/users/user.repository.impl.ts — Concrete implementation
import { Knex } from 'knex'
import { UserRepository } from './user.repository'
import { User } from './user.model'
import { DomainError, ErrorCode } from '../shared/errors/domain-error'

export class UserRepositoryImpl implements UserRepository {
  constructor(private readonly db: Knex) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db('users').where({ id, deleted_at: null }).first()
    return row ? User.fromRow(row) : null
  }

  async findByEmail(email: string): Promise<User | null> {
    const row = await this.db('users')
      .where({ email: email.toLowerCase(), deleted_at: null })
      .first()
    return row ? User.fromRow(row) : null
  }

  async findMany(params: { cursor?: string; limit: number }) {
    let query = this.db('users').whereNull('deleted_at').orderBy('id', 'asc').limit(params.limit + 1)

    if (params.cursor) {
      const decodedId = Buffer.from(params.cursor, 'base64').toString('utf8')
      query = query.where('id', '>', decodedId)
    }

    const rows = await query
    const hasMore = rows.length > params.limit
    const data = rows.slice(0, params.limit).map(User.fromRow)
    const lastId = data.at(-1)?.id
    const cursor = lastId ? Buffer.from(lastId).toString('base64') : null

    return { data, pagination: { cursor, hasMore } }
  }

  async save(user: User): Promise<User> {
    const row = user.toRow()
    await this.db('users')
      .insert(row)
      .onConflict('id')
      .merge(['email', 'name', 'updated_at', 'deleted_at'])
    return user
  }
}
```

---

## Domain Model Pattern

```typescript
// src/users/user.model.ts
// Pure domain entity — no I/O, no framework dependencies

import { randomUUID } from 'crypto'

export class User {
  readonly id: string
  email: string
  name: string
  passwordHash: string
  readonly createdAt: Date
  updatedAt: Date
  deletedAt: Date | null

  private constructor(props: UserProps) {
    this.id = props.id
    this.email = props.email
    this.name = props.name
    this.passwordHash = props.passwordHash
    this.createdAt = props.createdAt
    this.updatedAt = props.updatedAt
    this.deletedAt = props.deletedAt
  }

  static create(input: CreateUserInput): User {
    const now = new Date()
    return new User({
      id: randomUUID(),
      email: input.email.toLowerCase(),
      name: input.name.trim(),
      passwordHash: input.passwordHash,
      createdAt: now,
      updatedAt: now,
      deletedAt: null,
    })
  }

  static fromRow(row: UserRow): User {
    return new User({
      id: row.id,
      email: row.email,
      name: row.name,
      passwordHash: row.password_hash,
      createdAt: row.created_at,
      updatedAt: row.updated_at,
      deletedAt: row.deleted_at,
    })
  }

  update(input: UpdateUserInput): void {
    if (input.name !== undefined) this.name = input.name.trim()
    if (input.email !== undefined) this.email = input.email.toLowerCase()
    this.updatedAt = new Date()
  }

  softDelete(): void {
    this.deletedAt = new Date()
    this.updatedAt = new Date()
  }

  isDeleted(): boolean {
    return this.deletedAt !== null
  }

  toRow(): UserRow {
    return {
      id: this.id,
      email: this.email,
      name: this.name,
      password_hash: this.passwordHash,
      created_at: this.createdAt,
      updated_at: this.updatedAt,
      deleted_at: this.deletedAt,
    }
  }

  // Never expose passwordHash in serialized output
  toJSON() {
    return {
      id: this.id,
      email: this.email,
      name: this.name,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt,
    }
  }
}
```

---

## Middleware Chain

```typescript
// src/app.ts
// Register middleware in this exact order

export function createApp(deps: AppDependencies): Express {
  const app = express()

  // 1. Attach correlation ID to every request (before any logging)
  app.use(requestIdMiddleware())

  // 2. Request logging (uses request ID from step 1)
  app.use(requestLoggingMiddleware(deps.logger))

  // 3. CORS (before auth, so preflight OPTIONS requests succeed)
  app.use(corsMiddleware(deps.config.cors))

  // 4. Body parsing (before route handlers)
  app.use(express.json({ limit: '1mb' }))

  // 5. Rate limiting (after body parsing, before auth — to reject before doing expensive work)
  app.use(rateLimitMiddleware(deps.config.rateLimit))

  // 6. Health checks (no auth required — used by load balancer)
  app.use('/health', healthRouter(deps.db))

  // 7. Authentication (all routes below this are protected)
  app.use('/api', authMiddleware(deps.config.jwt))

  // 8. Route handlers
  app.use('/api/v1/users', createUserRouter(deps.userService))

  // 9. 404 handler (after all routes)
  app.use(notFoundMiddleware())

  // 10. Global error handler (must be last, must have 4 parameters)
  app.use(errorHandlerMiddleware(deps.logger))

  return app
}
```

**Auth middleware:**
```typescript
// Verify JWT, attach decoded payload to req.user, call next()
// On failure: respond 401 immediately, never call next()
function authMiddleware(jwtConfig: JwtConfig) {
  return (req: Request, res: Response, next: NextFunction) => {
    const token = req.headers.authorization?.replace('Bearer ', '')
    if (!token) return res.status(401).json(problemDetails(401, 'MISSING_TOKEN'))

    try {
      req.user = verifyToken(token, jwtConfig)
      next()
    } catch {
      return res.status(401).json(problemDetails(401, 'INVALID_TOKEN'))
    }
  }
}
```

**Global error handler:**
```typescript
// Must have exactly 4 params for Express to treat it as an error handler
function errorHandlerMiddleware(logger: Logger) {
  return (err: unknown, req: Request, res: Response, _next: NextFunction) => {
    if (err instanceof DomainError) {
      // Expected errors: log at WARN, return appropriate status
      logger.warn('Domain error', { code: err.code, message: err.message, requestId: req.id })
      return res.status(err.httpStatus).json(err.toProblemDetails())
    }

    // Unexpected errors: log full stack at ERROR, return generic 500
    logger.error('Unexpected error', { error: err, requestId: req.id, stack: (err as Error).stack })
    return res.status(500).json(problemDetails(500, 'INTERNAL_ERROR'))
  }
}
```

---

## Database Connection Handling

```typescript
// src/shared/database/database.ts

import Knex from 'knex'

export function createDatabase(config: DatabaseConfig): Knex {
  const db = Knex({
    client: 'postgresql',
    connection: {
      host: config.host,
      port: config.port,
      database: config.name,
      user: config.user,
      password: config.password,
      ssl: config.ssl ? { rejectUnauthorized: true } : false,
    },
    pool: {
      min: config.poolMin ?? 2,
      max: config.poolMax ?? 10,
      // Kill idle connections after 30 seconds
      idleTimeoutMillis: 30_000,
      // Fail fast if the pool is exhausted
      acquireTimeoutMillis: 5_000,
    },
    // Log slow queries (> 500ms) at WARN level
    asyncStackTraces: config.env === 'development',
  })

  return db
}

// Verify the connection is healthy at startup
export async function verifyDatabaseConnection(db: Knex): Promise<void> {
  await db.raw('SELECT 1')
}
```

---

## Configuration Loading Pattern

```typescript
// src/shared/config/config.ts

import { z } from 'zod'

const configSchema = z.object({
  // Server
  port: z.coerce.number().int().min(1).max(65535).default(3000),
  env: z.enum(['development', 'test', 'production']).default('development'),
  serviceName: z.string().default('my-api'),
  version: z.string().default('unknown'),
  shutdownTimeoutMs: z.coerce.number().default(10_000),

  // Database
  database: z.object({
    host: z.string(),
    port: z.coerce.number().default(5432),
    name: z.string(),
    user: z.string(),
    password: z.string(),
    poolMin: z.coerce.number().default(2),
    poolMax: z.coerce.number().default(10),
    ssl: z.coerce.boolean().default(false),
  }),

  // Auth
  jwt: z.object({
    secret: z.string().min(32),
    expiresInSeconds: z.coerce.number().default(3600),
    refreshExpiresInSeconds: z.coerce.number().default(604800),
  }),

  // Rate limiting
  rateLimit: z.object({
    windowMs: z.coerce.number().default(60_000),
    max: z.coerce.number().default(100),
  }),
})

function loadConfig() {
  const result = configSchema.safeParse({
    port: process.env.PORT,
    env: process.env.NODE_ENV,
    serviceName: process.env.SERVICE_NAME,
    version: process.env.npm_package_version,
    shutdownTimeoutMs: process.env.SHUTDOWN_TIMEOUT_MS,
    database: {
      host: process.env.DB_HOST,
      port: process.env.DB_PORT,
      name: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
    },
    jwt: {
      secret: process.env.JWT_SECRET,
      expiresInSeconds: process.env.JWT_EXPIRES_IN_SECONDS,
    },
    rateLimit: {
      windowMs: process.env.RATE_LIMIT_WINDOW_MS,
      max: process.env.RATE_LIMIT_MAX,
    },
  })

  if (!result.success) {
    console.error('Invalid configuration:', result.error.format())
    process.exit(1)  // Fail fast with a useful error message
  }

  return result.data
}

// Singleton: parse once at module load time
export const config = loadConfig()
```

---

## Health Check Endpoint Implementation

```typescript
// src/health/health.handler.ts

import { Router } from 'express'
import { Knex } from 'knex'

export function healthRouter(db: Knex): Router {
  const router = Router()

  // Liveness probe: Is the process running and not deadlocked?
  // Kubernetes calls this to decide if the pod should be restarted.
  // Return 200 immediately — do NOT check dependencies here.
  router.get('/live', (_req, res) => {
    res.status(200).json({ status: 'ok' })
  })

  // Readiness probe: Is the service ready to accept traffic?
  // Kubernetes calls this to decide if the pod should receive traffic.
  // Check all critical dependencies.
  router.get('/ready', async (_req, res) => {
    const checks: Record<string, 'ok' | 'error'> = {}
    let isReady = true

    try {
      await db.raw('SELECT 1')
      checks.database = 'ok'
    } catch {
      checks.database = 'error'
      isReady = false
    }

    res.status(isReady ? 200 : 503).json({
      status: isReady ? 'ready' : 'degraded',
      checks,
      version: process.env.npm_package_version,
      timestamp: new Date().toISOString(),
    })
  })

  return router
}
```

---

## Graceful Shutdown Pattern

```
Shutdown sequence:

1. Receive SIGTERM / SIGINT
2. Stop accepting new connections (server.close())
3. Wait for in-flight requests to complete
4. Close database connection pool
5. Close message queue connections
6. Flush log buffers
7. Exit with code 0

Force-kill timeout: if step 3 takes longer than SHUTDOWN_TIMEOUT_MS (default: 10s),
log an error and exit with code 1 to let the orchestrator restart the pod.
```

---

## Complete Example: User Authentication API

Implements: registration, login (issue JWT), token refresh, logout, get current user.

**Directory structure:**
```
src/
  auth/
    auth.service.ts
    auth.handler.ts
    auth.validator.ts
    token.repository.ts
    token.repository.impl.ts     # Redis-backed refresh token store
  users/
    user.model.ts
    user.service.ts
    user.repository.ts
    user.repository.impl.ts      # PostgreSQL
    user.handler.ts
```

**POST /api/v1/auth/register:**
```
Input:  { email, password, name }
Validate: email format, password strength (min 10 chars, 1 number, 1 special), name non-empty
Action:
  1. Check no existing user with this email (409 if exists)
  2. Hash password with bcrypt (cost factor 12)
  3. Create User domain object
  4. Save to database
  5. Issue access token (JWT, 1 hour) and refresh token (opaque, 7 days)
  6. Store refresh token hash in Redis with TTL = 7 days, keyed by userId
Output: 201 { user: { id, email, name }, tokens: { accessToken, refreshToken, expiresAt } }
```

**POST /api/v1/auth/login:**
```
Input:  { email, password }
Action:
  1. Find user by email (return 401 "invalid credentials" if not found — do NOT say "user not found")
  2. Verify password against hash (return same 401 if wrong — no oracle)
  3. Invalidate existing refresh tokens for this user
  4. Issue new access token and refresh token
Output: 200 { user: { id, email, name }, tokens: { accessToken, refreshToken, expiresAt } }
```

**POST /api/v1/auth/refresh:**
```
Input:  { refreshToken }
Action:
  1. Decode refresh token to extract userId
  2. Look up stored refresh token hash in Redis
  3. If not found or hash does not match: 401 (token invalid or already rotated)
  4. Delete old refresh token from Redis (rotation — one-time use)
  5. Issue new access token and refresh token
  6. Store new refresh token hash in Redis
Output: 200 { tokens: { accessToken, refreshToken, expiresAt } }
```

**POST /api/v1/auth/logout:**
```
Input:  Bearer token in Authorization header
Action:
  1. Extract userId from verified JWT
  2. Delete all refresh tokens for this userId from Redis
Output: 204 No Content
```

**GET /api/v1/users/me:**
```
Auth:   Required (Bearer JWT)
Action:
  1. Extract userId from req.user (set by auth middleware)
  2. Fetch user from database
Output: 200 { data: { id, email, name, createdAt } }
```

**Security properties of this design:**
- Password never stored or logged in plaintext
- Login errors do not reveal whether the email exists
- Refresh tokens are opaque (not JWTs) and stored as hashes — a leaked token DB is not immediately exploitable
- Refresh token rotation means a stolen refresh token can only be used once before being invalidated
- Logout invalidates all sessions for the user globally
