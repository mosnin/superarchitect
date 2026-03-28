# Engineering Standards

Canonical engineering standards for the SuperArchitect Agentic OS. Every Engineer agent implementation must conform to these standards. Deviations require an explicit ADR entry with rationale.

---

## Code Structure Standards

### File Organization

Prefer **feature-based (vertical slice) organization** for application code. Group everything that belongs to a feature together: handler, service, repository, models, tests, and types for that feature live in one directory.

```
src/
  users/
    user.model.ts
    user.service.ts
    user.repository.ts
    user.handler.ts
    user.types.ts
    user.service.test.ts
    user.repository.test.ts
    user.handler.test.ts
  orders/
    order.model.ts
    ...
  shared/
    database/
    middleware/
    errors/
    logging/
    config/
```

Use **layer-based organization** only for libraries, SDKs, or infrastructure packages where a vertical slice provides no natural boundary.

**Rationale:** Feature-based organization makes it easy to find everything related to a concern, allows features to be deleted cleanly, and reduces the cognitive overhead of cross-directory navigation.

### Module Boundaries

Modules communicate only through their public interface. Internal implementation files are not imported from outside the module.

Rules:
- Each feature module exports exactly one index file (`index.ts`, `__init__.py`, etc.)
- Imports between feature modules go through the exported index, never to internal files
- Circular dependencies between modules are prohibited
- Shared utilities live in `shared/` and may be imported by any module
- Infrastructure code (database, queue clients) is injected, not imported directly by domain code

### Import/Dependency Rules

Order imports in this sequence (enforce with linter):
1. Standard library imports
2. Third-party library imports
3. Internal shared/common imports
4. Feature-local imports

Rules:
- No wildcard imports (`import * from`) in application code
- No side-effect-only imports except in entry points and test setup files
- No relative path imports that traverse more than two directory levels up (`../../..` is a smell that the module boundary is wrong)
- Type-only imports use `import type` syntax where the language supports it

### Configuration Management

All runtime configuration is externalized. The application has no hardcoded environment-specific values.

Configuration loading order (highest to lowest priority):
1. Environment variables
2. `.env.local` (developer override, gitignored)
3. `.env.<environment>` (e.g., `.env.production`)
4. `.env` (base defaults)
5. Default values in code (only for non-critical optional settings)

Configuration schema:
- All expected environment variables are documented in `.env.example` with descriptions and example values
- Configuration is parsed and validated at startup. If required variables are missing or malformed, the process exits immediately with a clear error message
- Configuration is loaded once into a typed config object and injected — never call `process.env` or `os.environ` in business logic

---

## API Design Standards

### REST API Design Rules

**Resource Naming:**
- Resources are nouns, plural: `/users`, `/orders`, `/payment-methods`
- Use kebab-case for multi-word resources: `/shipping-addresses`, `/api-keys`
- Nested resources express ownership: `/users/{userId}/orders`
- Limit nesting to two levels. Deeper nesting signals a missing top-level resource
- Never use verbs in resource paths: use `/orders/{id}/cancel` only when there is no cleaner resource model (`PATCH /orders/{id}` with `{ status: "cancelled" }` is usually better)

**HTTP Methods:**
| Method | Semantics | Idempotent | Safe |
|---|---|---|---|
| GET | Retrieve a resource or collection | Yes | Yes |
| POST | Create a resource or trigger an action | No | No |
| PUT | Replace a resource entirely | Yes | No |
| PATCH | Partially update a resource | No | No |
| DELETE | Remove a resource | Yes | No |

- POST to a collection creates a new resource: `POST /users` → `201 Created` with `Location` header
- PUT replaces the entire resource. If you only support partial updates, use PATCH
- DELETE returns `204 No Content` on success, `404 Not Found` if the resource does not exist
- Repeated DELETE on an already-deleted resource returns `404`, not `204` (idempotency applies to the effect on server state, not the response code)

**HTTP Status Codes:**
| Code | Meaning | When to use |
|---|---|---|
| 200 | OK | Successful GET, PATCH, PUT |
| 201 | Created | Successful POST creating a resource |
| 202 | Accepted | Request accepted for async processing |
| 204 | No Content | Successful DELETE or PUT/PATCH with no response body |
| 400 | Bad Request | Malformed request syntax, invalid parameters |
| 401 | Unauthorized | Missing or invalid authentication credentials |
| 403 | Forbidden | Authenticated but not authorized for this resource |
| 404 | Not Found | Resource does not exist |
| 409 | Conflict | State conflict (duplicate creation, optimistic lock failure) |
| 422 | Unprocessable Entity | Syntactically valid but semantically invalid input |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Upstream dependency failure |
| 503 | Service Unavailable | Service is down or overloaded |

**Pagination:**
Prefer cursor-based pagination for large, frequently-updated datasets.

Cursor pagination response:
```json
{
  "data": [...],
  "pagination": {
    "cursor": "eyJpZCI6MTAwfQ==",
    "hasMore": true,
    "limit": 20
  }
}
```

Offset pagination (acceptable for smaller, static datasets):
```json
{
  "data": [...],
  "pagination": {
    "total": 342,
    "page": 2,
    "pageSize": 20,
    "pages": 18
  }
}
```

Default page size: 20. Maximum page size: 100. Enforce the maximum server-side.

**Filtering and Sorting:**
- Filtering via query parameters: `GET /orders?status=pending&customerId=123`
- Multi-value filters use repeated params: `GET /orders?status=pending&status=processing`
- Sorting: `GET /users?sort=createdAt:desc,lastName:asc`
- Date range: `GET /events?from=2024-01-01T00:00:00Z&to=2024-01-31T23:59:59Z`
- All dates in ISO 8601 UTC format

**API Versioning:**
Use URL versioning: `/api/v1/users`, `/api/v2/users`
- Increment the major version when making breaking changes
- Maintain at least one previous major version for a documented deprecation period
- Communicate deprecation via `Deprecation` and `Sunset` response headers
- Never remove a version without a migration path published to consumers

### Error Response Format (RFC 7807 Problem Details)

All API errors return a Problem Details object:

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The request body contains invalid fields.",
  "instance": "/api/v1/users/create#request-id-abc123",
  "errors": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Must be a valid email address"
    },
    {
      "field": "age",
      "code": "OUT_OF_RANGE",
      "message": "Must be between 0 and 150"
    }
  ]
}
```

- `type`: A URI identifying the error type (should resolve to documentation)
- `title`: Human-readable short description, stable across occurrences
- `status`: HTTP status code (must match the response status code)
- `detail`: Human-readable explanation specific to this occurrence
- `instance`: URI reference to this specific error occurrence
- `errors`: Extension field for validation errors with field-level detail

Content-Type for error responses: `application/problem+json`

### Rate Limiting Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 842
X-RateLimit-Reset: 1706745600
Retry-After: 60          (only on 429 responses)
```

### Authentication/Authorization Patterns

- Authentication: Bearer JWT tokens in `Authorization: Bearer <token>` header
- Token validation: Verify signature, expiry, issuer, and audience
- Service-to-service: Mutual TLS or short-lived JWT signed by a service account
- API keys: Hash the key before storing; compare hashes, never plaintext
- Authorization: Attribute-Based Access Control (ABAC) for complex permission models; Role-Based (RBAC) for simpler models
- Check authorization at the handler layer, before calling the service layer
- Never expose resource existence to unauthorized callers: return `404` (not `403`) when the caller should not know whether the resource exists

### GraphQL Design Rules

- Use schema-first design: write the SDL before the resolvers
- Every field that can be null must be explicitly nullable in the schema — don't default to nullable for convenience
- Use the Connection pattern (Relay spec) for all paginated collections
- Mutations return a result type with both success data and errors, never just the data
- Use DataLoader for all resolver-level database lookups to prevent N+1
- Rate-limit by query complexity score, not just request count
- Disable introspection in production unless explicitly needed by consumers

### gRPC Design Rules

- Use proto3 syntax
- Namespace all services and messages under a versioned package: `myapp.v1`
- Every RPC method returns a response message — never use `google.protobuf.Empty` as a response for mutation operations
- Use server-streaming for collection responses that may be large
- Implement health check using the standard `grpc.health.v1.Health` service
- Always set deadlines on outgoing gRPC calls
- Use interceptors for logging, authentication, and panic recovery

---

## Database Standards

### Schema Design Principles

- Every table has a surrogate primary key: UUID v7 (time-sortable) preferred, auto-increment integer acceptable
- Every table has `created_at` and `updated_at` timestamp columns with timezone
- Soft delete: add `deleted_at` nullable timestamp column. Never delete business records. Filter `WHERE deleted_at IS NULL` in the repository layer
- Foreign keys are always declared with explicit referential integrity actions (`ON DELETE RESTRICT` by default; `ON DELETE CASCADE` only when child rows have no meaning without the parent)
- No polymorphic foreign keys (a foreign key that references different tables depending on a type column) — use table-per-type inheritance or a join table instead
- Column constraints are defined in the database schema, not only in application code: `NOT NULL`, `CHECK`, `UNIQUE`
- Avoid nullable columns unless the absence of a value has distinct meaning from a default value

### Migration Strategy

- All schema changes are implemented as versioned migration files
- Migration files are numbered sequentially and never modified after they are committed
- Every migration has an `up` and a `down` method
- Migrations are applied automatically on deployment
- No DDL changes (ALTER TABLE, CREATE INDEX CONCURRENTLY) are made manually against production
- Large table migrations (adding indexes, backfilling columns) use online/non-blocking DDL operations or multi-step migrations

### Index Strategy

Mandatory indexes:
- Primary key (automatic)
- All foreign key columns
- All columns used in `WHERE` clauses of common queries
- All columns used in `ORDER BY` clauses on paginated queries
- Composite indexes for common multi-column filter combinations

Index hygiene:
- Unused indexes are removed (they slow writes and consume storage)
- Partial indexes are used when a query consistently filters on a specific value (e.g., `WHERE deleted_at IS NULL`)
- Expression indexes are used for case-insensitive search, JSON field extraction, etc.
- Index creation on large tables uses `CREATE INDEX CONCURRENTLY` (PostgreSQL) or equivalent to avoid table locks

### Query Optimization Rules

- All queries must have an `EXPLAIN ANALYZE` verified to use index scans (not sequential scans) on tables over 10,000 rows
- `SELECT *` is prohibited in application code — select only the columns you need
- Avoid functions in `WHERE` clauses on indexed columns: `WHERE lower(email) = ?` prevents index use unless there is a functional index
- Avoid wildcard prefix LIKE queries (`WHERE name LIKE '%smith'`): they are not indexable
- `COUNT(*)` on large tables is expensive; maintain counts in a separate denormalized counter when real-time count is needed

### Transaction Boundaries

- Transactions enclose all operations that must succeed or fail atomically
- Transactions are owned by the service layer, not the repository layer
- Repositories accept an optional transaction context and participate if one is provided
- Transactions are kept as short as possible; never hold a transaction open across a network call
- Use optimistic locking (`version` column) for concurrent update conflicts; use pessimistic locking (`SELECT FOR UPDATE`) only when optimistic locking is impractical

### N+1 Prevention

- In ORM-based code, use eager loading for all related entities accessed in a loop
- In GraphQL resolvers, use DataLoader for all database lookups
- In REST handlers that return collections, join related data in the query rather than fetching it per item
- Integration tests should assert on query count to catch N+1 regressions

---

## Testing Standards

### Test Pyramid

Target distribution for most services:
- **Unit tests**: 70% — Fast, isolated, test a single function or class
- **Integration tests**: 25% — Test that components work together (real database, real queue)
- **End-to-end tests**: 5% — Test critical user journeys through the deployed system

### What to Unit Test

- All domain model methods and business rules
- All service layer methods (using mocked repositories)
- All pure transformation/computation functions
- All validation logic
- Error handling paths (not just the happy path)

### What to Integration Test

- Repository implementations (against a real or containerized database)
- API handlers (against a real HTTP server with real dependencies)
- Queue consumers (against a real or containerized message broker)
- External service adapters (against a mock server or test environment)

### Test Naming Conventions

Use the `given_when_then` or `describe_context_it` pattern. Test names must read as complete sentences describing the expected behavior:

```
GOOD:
  "returns 404 when the user does not exist"
  "sends a confirmation email when an order is placed"
  "retries up to 3 times before marking a job as failed"

BAD:
  "test user service"
  "it works"
  "error case"
```

### Fixture and Factory Patterns

- Use factories (not fixtures) to generate test data: factories produce valid objects with defaults that can be overridden per test
- Factories use realistic but non-sensitive fake data (faker libraries)
- Database fixtures (pre-seeded data) are acceptable for read-only reference data; avoid them for mutable test data
- Each test creates and cleans up its own data — do not share mutable state between tests

### Mocking Strategy

- Mock at the boundary of your module: mock the repository interface, not the database driver
- Use explicit dependency injection so that dependencies can be replaced with mocks in tests
- Prefer stub/fake implementations over mock frameworks for complex objects (a `FakeUserRepository` that stores data in memory is more reliable than a deeply configured mock)
- Do not mock types you do not own (mock your adapter/wrapper around the third-party type, not the type itself)
- Verify that mocks are called with the right arguments for behavior you care about; do not verify call counts on internal implementation details

### Coverage Expectations

Target: **80% line coverage** for application code (service and domain layers: higher; infrastructure adapters: lower is acceptable because they are integration-tested).

100% coverage is not the goal because:
- It incentivizes testing implementation instead of behavior
- It encourages trivial tests that inflate the number without increasing confidence
- Infrastructure code that is tested via integration tests shows low unit-test coverage — that is correct

Coverage is a minimum bar, not a quality metric. A test suite with 95% coverage but no assertions on error paths is worse than one with 75% coverage that exercises all meaningful behavior.

---

## Documentation Standards

### Inline Documentation Rules

**Comment the "why", not the "what":**
```
BAD:  // Increment counter by 1
      counter++

GOOD: // Rate limiter uses a sliding window; we increment before checking the limit
      // so that concurrent requests share the same window accurately.
      counter++
```

Document these:
- Non-obvious algorithmic choices with a reference to the source or paper
- Business rule implementations with a reference to the requirement or ticket
- Workarounds for bugs in libraries or platforms, with issue tracker links
- Performance trade-offs (why you chose this less-obvious approach)

Do not document:
- What the code obviously does
- Standard language idioms that any competent developer knows
- Variable types (use type annotations instead)

### API Documentation Format

- REST APIs: OpenAPI 3.1 spec, auto-generated from code annotations where possible, stored in `openapi.yaml`
- gRPC: Self-documenting via `.proto` file comments; generate HTML docs with protoc-gen-doc
- Every endpoint documents: description, all parameters (path, query, header, body), all response codes with example bodies, authentication requirements, rate limit tier

### Architecture Decision References

When an implementation reflects a non-obvious architectural decision, reference the ADR number:
```
# This service uses eventual consistency for the order count cache.
# See ADR-012: Order Count Caching Strategy
```

---

## Git Standards

### Commit Message Format (Conventional Commits)

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
| Type | Meaning |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `docs` | Documentation only |
| `chore` | Build process, dependency updates, tooling |
| `ci` | CI/CD configuration |

**Rules:**
- Subject line: imperative mood, no period, max 72 characters: "add user authentication endpoint" not "Added user authentication endpoint."
- Body: Explain the why, not the what. Wrap at 72 characters.
- Footer: Reference issues (`Closes #123`, `Fixes #456`), note breaking changes (`BREAKING CHANGE: removes v1 API`)

**Examples:**
```
feat(auth): add JWT refresh token rotation

Implements sliding session expiry using refresh token rotation.
Each refresh invalidates the previous token and issues a new pair.
This prevents token theft from stale refresh tokens.

Closes #284
```

```
fix(orders): prevent duplicate order creation on retry

The order creation endpoint was not idempotent. Concurrent retries
from the client could create multiple orders for the same cart.
Added an idempotency key check using a Redis lock with a 60-second TTL.

BREAKING CHANGE: clients must now include Idempotency-Key header
Closes #391
```

### Branch Naming

```
<type>/<ticket-id>-<short-description>

feat/AUTH-284-refresh-token-rotation
fix/ORD-391-prevent-duplicate-orders
refactor/PERF-102-query-optimization
chore/update-dependencies-2024-q1
```

- Use hyphens, not underscores
- Keep the description short (3-5 words)
- Always reference a ticket ID for feature and fix branches

### PR Structure

**Title:** Same format as a commit subject: `feat(auth): add JWT refresh token rotation`

**Description template:**
```markdown
## What
[1-3 sentences describing what changed]

## Why
[1-3 sentences on the motivation — link to ticket/spec]

## How
[Key implementation decisions worth highlighting]

## Testing
[How you tested this; what CI gates cover it]

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No secrets committed
- [ ] Linter and formatter pass
- [ ] Migration is backward compatible (or migration plan documented)
```

### PR Review Checklist

Reviewers verify:
- [ ] The change does what the description says
- [ ] Business logic is correct (not just syntactically valid)
- [ ] Error handling is complete
- [ ] Tests cover the meaningful behavior, including edge cases
- [ ] No N+1 queries introduced
- [ ] No secrets or sensitive data committed
- [ ] API changes are backward compatible or versioned
- [ ] Performance implications are acceptable
- [ ] The code is understandable to someone unfamiliar with this area
