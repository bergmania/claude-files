# Claude Code Guidelines

## Language

- Respond in **Danish** for conversations and explanations
- Use **American English** for all artifacts others will read:
  - Commit messages
  - PR descriptions
  - Code comments
  - Documentation

## Communication Style

- **Think thoroughly** before responding - consider implications, edge cases, and alternatives
- **Ask clarifying questions** before providing detailed answers
- Understand the context and requirements before diving into implementation details
- When in doubt, ask rather than assume

### Uncertainty

- Always communicate uncertainty with a **confidence score (1-100)**
- If confidence is **below 85**, ask clarifying questions before proceeding
- Be explicit about assumptions and unknowns

## Output Formats

### Diagrams

- Use **Mermaid** for all diagrams (flowcharts, sequence diagrams, ERDs, etc.)
- Wrap Mermaid code in ```mermaid code blocks

### Rich Text

- Use **Markdown** for all rich text formatting
- Use appropriate heading levels, lists, tables, and code blocks

### Code Changes

- Show only the **changed/relevant sections**, not entire files
- Prefer **diff format** when showing modifications
- Always explain **why** the change is made
- Mention **alternatives** when relevant, with brief pros/cons
- **Proactively suggest improvements** when spotted (but keep them separate from the main task)

### Testing

- **Always create blackbox tests** for new/changed code
- Choose the appropriate test level:
  - **Unit tests** for isolated logic
  - **Integration tests** for application services/use cases
  - **E2E tests** for API endpoints

### Trade-offs

When making decisions, prioritize:
- **Readability** over cleverness
- **Flexibility** over rigidity
- Explain trade-offs when deviating from these defaults

### Breaking Changes

- Communicate breaking changes at the **OpenAPI/openapi.json level**
- Clearly document what changed and migration path

### Documentation

- **Document all public types** (classes, interfaces, methods, properties)
- Keep documentation in sync with code changes

### Dependencies

- External packages are allowed, but **only in Infrastructure projects**
- **Exception:** `Microsoft.Extensions.*` packages are allowed in all layers (these are foundational .NET abstractions)
- **Always ask for approval** before adding new dependencies
- Prefer well-maintained, widely-used packages

### Error Messages & Logging

- **User-facing errors**: Clear, friendly, actionable - always in English
- **Log messages**: Technical, detailed, with context - always in English
- Include correlation IDs and relevant data in logs

### Technical Debt

- **Proactively identify** technical debt when spotted
- Suggest whether it warrants a **separate work item/task**
- Don't fix unrelated tech debt without asking

### Performance

- **Always consider performance for hot paths**
- Profile before optimizing non-critical paths
- Document performance-critical sections

### Security

- **Always check for security issues** using OWASP Top 10 as baseline
- Flag potential vulnerabilities immediately
- Suggest secure alternatives when spotting risky patterns

## Tech Stack

- **Backend:** C# (.NET latest)
- **Frontend:** TypeScript with Lit (native web components)
- **Database:** PostgreSQL (or MSSQL where specified)
- **ORM:** Entity Framework Core (EF Core)
- **JSON:** System.Text.Json
- **API Clients:** Kiota for generating strongly-typed OpenAPI clients
- **Orchestration:** .NET Aspire
- **Package Management:** Central Package Management (Directory.Packages.props)

## Code Style & Conventions

### Naming

- Follow standard **C# naming conventions**
- Use `I` prefix for interfaces (`IUserRepository`, `IEmailService`)
- Use `Async` suffix for async methods

### File & Namespace Organization

- **One class per file**
- Use **file-scoped namespaces** (always)
- Namespaces must follow folder structure
- Extension methods must use the **same namespace as the class they extend**
- Only use default global usings (no custom global usings)

### Project Structure

- Use **feature-based** folder structure (not layer-based)
- Group related files by feature/domain concept, not by technical layer

### Immutability

- Prefer **immutability** by default
- Use `record` for DTOs, value objects, and data carriers
- Use `init` and `required` properties for immutable classes
- Primary constructors only in `record` types (not in classes)

### Class Design

- Mark classes as `sealed` by default (better performance, explicit inheritance)
- Use **collection expressions** (`[1, 2, 3]`) for collection initialization

### Date & Time

- **Never** use `DateTime` - use `DateTimeOffset` for timestamps
- Use `DateOnly` for dates without time
- Use `TimeOnly` for times without date
- Always inject `TimeProvider` instead of using `DateTime.Now` / `DateTimeOffset.Now`

### Nullability

- **Nullable reference types** must be enabled
- Use **Option/Maybe pattern** for representing absence of value (not null)
- Avoid returning or accepting null where possible

### Documentation

- Add **XML comments** on all public APIs (classes, methods, properties)
- XML comments flow through to OpenAPI/Swagger documentation
- Focus on "why" and usage, not restating the obvious

## Architecture Principles

Follow **Clean Architecture**:

- Domain layer has zero external dependencies
- Application layer depends only on Domain
- Infrastructure layer contains all external dependencies (databases, APIs, file systems)
- Never add NuGet package references outside Infrastructure projects unless absolutely necessary
  - **Exception:** `Microsoft.Extensions.*` packages (DI abstractions, logging, options, etc.) are allowed everywhere

### Fail-Fast & Error Handling

- Follow the **fail-fast** principle: detect and report errors as early as possible
- Use **value types** (strongly-typed wrappers) to validate input at construction time
- Use the **Result pattern** to communicate failures explicitly without exceptions
  - Return `Result<T>` or `Result<T, TError>` instead of throwing exceptions for expected failure cases
  - Reserve exceptions for truly exceptional/unexpected situations
- Validate at system boundaries and let invalid data fail immediately
- Validation logic belongs in **value types** or **request/command models**
- Use **DataAnnotations** for validation

**Validation responsibility by layer:**

| Validation Type | Where | Example |
|-----------------|-------|---------|
| Format/syntax | Value type constructor | Email format, phone number pattern |
| Required fields | Request/command model | DataAnnotations `[Required]` |
| Business rules | Use case | Email uniqueness, account balance |
| Cross-field | Use case or model | Date range validity |

### Async & CancellationToken

- All I/O-bound code must be **async**
- Always accept and propagate `CancellationToken` through the call chain
- Use `CancellationToken` parameter (injected or passed) in all async methods

### Dependency Injection

- Use **constructor injection** exclusively (no property or method injection)
- All dependencies must be injected, including logging (`ILogger<T>`)
- Use `IOptions<T>` for configuration values
- Register services with appropriate lifetimes (Scoped for request-bound, Singleton for stateless)

### Logging

- Use **structured logging** with semantic log templates
- Inject `ILogger<T>` via constructor
- Include relevant context (IDs, correlation) in log messages
- Use appropriate log levels (Debug, Information, Warning, Error)
- Use **high-performance logging** with `[LoggerMessage]` source generators for hot paths
  - Avoids boxing and string allocations
  - Define log methods as `partial` with `[LoggerMessage]` attribute

### Mapping

- Use **manual mapping** (no AutoMapper, Mapperly, or similar)
- Keep mapping logic explicit and co-located with the types being mapped
- Use extension methods or static factory methods for mapping
- **Mappers must be pure** - no service dependencies allowed
- If service dependencies are needed, use a **Factory** instead (not a Mapper)

## Frontend (TypeScript)

### Web Components with Lit

- Use **Lit** for building native web components
- Follow web component standards and best practices
- Keep components small and focused on single responsibility
- Use Shadow DOM for style encapsulation

### API Client Generation

- API clients are auto-generated from OpenAPI specs via npm scripts
- Never manually write API client code - regenerate when API changes
- Generated clients live in a dedicated folder (do not modify generated files)

## API Design

Follow **REST** principles and **OpenAPI** specification:

- Use proper HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Use resource-based URLs (`/customers/{id}` not `/getCustomer`)
- Return appropriate HTTP status codes (200, 201, 204, 400, 404, 409, etc.)
- Use plural nouns for collection endpoints (`/orders` not `/order`)
- Support filtering and sorting for collection endpoints
- Version APIs when breaking changes are necessary
- All endpoints must be documented with OpenAPI/Swagger annotations
- Use consistent response formats across all endpoints

### Pagination

- Endpoints that can return many results with few parameters **must** support pagination
  - Example: `GET /organizations` requires pagination
  - Example: `GET /organizations?ids=id1,id2,id3` does **not** require pagination (explicit IDs limit results)
- Use `skip` and `take` parameters (not `pageNumber`/`pageIndex` or `pageSize`)
- Return total count in response to enable client-side pagination UI

### Error Handling (Problem Details RFC 7807)

Use ASP.NET Core's built-in Problem Details for all error responses:

- Return `ProblemDetails` for 4xx/5xx responses
- Use `ValidationProblemDetails` for validation errors (400)
- Include `traceId` in extensions for debugging
- Map domain errors to appropriate HTTP status codes consistently

## Quality Requirements

### Concurrency

All code runs in load-balanced web applications. Always consider:

- Race conditions between concurrent requests
- Optimistic concurrency with row versioning where appropriate
- Database-level locking strategies when needed
- Idempotency for operations that may be retried

**Optimistic concurrency implementation:**

- Use `ETag` response header for versioning
- `If-Match` request header is **optional**:
  - If provided: validate against current `ETag` and return `409 Conflict` on mismatch
  - If omitted: update blindly without concurrency check (last-write-wins)
- For PostgreSQL with EF Core:
  - Use `xmin` system column as concurrency token (automatically updated on row changes)
  - Add `LastModified` timestamp on aggregate roots for client-visible versioning

### Security

- Validate all input
- Use parameterized queries (or EF Core where possible)
- Apply principle of least privilege
- Consider OWASP top 10

### Performance

- Avoid N+1 queries
- Use appropriate indexes
- Consider caching strategies
- Profile before optimizing

## Database & EF Core

### Migrations

- Use **code-first migrations** with EF Core
- Migrations are committed to source control
- Each migration should be atomic and reversible when possible
- Name migrations descriptively (e.g., `AddUserEmailIndex`, `CreateOrdersTable`)

## Rate Limiting

Use ASP.NET Core's built-in rate limiting:

- Configure rate limiting policies for API endpoints
- Use appropriate algorithms (fixed window, sliding window, token bucket)
- Return `429 Too Many Requests` with `Retry-After` header
- Consider per-user vs per-IP vs global limits based on endpoint

## Caching

When caching is needed, use .NET 9's `HybridCache`:

- Combines in-memory (L1) and distributed (L2) caching
- Use appropriate cache durations based on data volatility
- Always consider cache invalidation strategy
- Include cache key versioning for safe deployments

## Health Checks

Use ASP.NET Core's built-in health check framework:

- Add health checks for all external dependencies (database, APIs, etc.)
- Use `/health` for liveness (is the app running?)
- Use `/health/ready` for readiness (can the app serve traffic?)
- Include `AspNetCore.HealthChecks.*` packages for common dependencies
- Configure Aspire to use health checks for orchestration

## Observability

### OpenTelemetry (via Aspire)

- Aspire configures OpenTelemetry automatically
- All traces, metrics, and logs flow through Aspire dashboard
- Use `Activity` for custom spans when needed

### Correlation

- Use `TraceId` for correlating requests across services
- Include `TraceId` in all log messages (automatic via structured logging)
- Return `TraceId` in Problem Details responses for debugging
- Pass `TraceId` to external service calls via headers

## Azure Boards Work Items

Use the standard Agile hierarchy: **Epic → Feature → PBI → Task**

| Type | Purpose | Scope | Example |
|------|---------|-------|---------|
| **Epic** | Roadmap item visible to stakeholders | Spans multiple sprints/quarters | "Partner Portal v2" |
| **Feature** | Concrete deliverable that provides user value | Typically 1-3 sprints | "Partner can view their customers" |
| **PBI** | User story or specific requirement | Fits within a sprint | "As a partner, I can filter customers by status" |
| **Task** | Technical implementation detail | Hours to days | "Add filtering endpoint", "Update UI component" |

**Guidelines:**

- Epics should be meaningful to stakeholders (not technical jargon)
- Features should be demonstrable and deliver standalone value
- PBIs should follow user story format when possible: "As a [role], I can [action] so that [benefit]"
- Tasks are optional - only create them when a PBI needs to be broken down for tracking
- Keep descriptions concise; use acceptance criteria for details
- Link related work items (parent/child, related, predecessor)
- **Bugs are tracked as PBIs** (not separate Bug work item type)

### Estimation

Use **Story Points** with Fibonacci scale: 1, 2, 3, 5, 8, 13

| Points | Complexity | Example |
|--------|------------|---------|
| 1 | Trivial | Typo fix, config change |
| 2 | Simple | Small isolated change, clear solution |
| 3 | Moderate | Standard feature, some unknowns |
| 5 | Complex | Multiple components, integration work |
| 8 | Large | Significant effort, consider splitting |
| 13 | Too big | **Must be split** into smaller PBIs |

- Estimate relative complexity, not time
- If unsure between two values, pick the higher one
- PBIs larger than 8 points should typically be split

### Definition of Ready (DoR)

A PBI is ready for sprint when:

- [ ] Title clearly describes the outcome
- [ ] Description/user story is complete
- [ ] Acceptance criteria are defined and testable
- [ ] Dependencies are identified and resolved (or planned)
- [ ] Estimated in story points
- [ ] Small enough to complete within a sprint
- [ ] Team has enough information to start work

## Testing Requirements

### Test Class Naming

- Test classes must be postfixed with `Tests` (e.g., `OrderServiceTests`)
- One test class per class under test

### Test Method Naming

Use the pattern: `MethodName_Scenario_ExpectedResult`

Examples:
- `CreateOrder_WithValidInput_ReturnsCreatedOrder`
- `CreateOrder_WithDuplicateId_ReturnsConflict`
- `GetOrders_WithPagination_ReturnsPagedResult`
- `DeleteOrder_WhenNotFound_ReturnsNotFound`

### Unit Tests

- Optional, but must be blackbox tests so refactoring does not break tests

### Integration Tests

- Every use case / application service must have integration tests
- Tests run against real database (containerized via Aspire)
- Test the full flow from application layer through infrastructure

### E2E Tests

- Every API endpoint must have E2E tests
- Use Aspire to spin up the complete environment
- Test realistic scenarios including error cases

### Testing ID Collections

- When testing endpoints/methods that accept a list o

## Code Review

When reviewing a PR, always provide a structured assessment with scores.

### Scoring

- Rate every category on a **1-100 scale**
- Provide an **overall score** (weighted by relevance, not a simple average)
- Scores are advisory — no automatic approve/reject thresholds

### Categories

**Always include these fixed categories:**

| Category | Focus Areas |
|----------|------------|
| **Security** | OWASP top 10, input validation, auth/authz, secrets exposure |
| **Performance** | N+1 queries, caching, hot paths, async patterns, allocations |
| **Architecture** | Clean Architecture, SOLID, separation of concerns, dependency direction |
| **Consistency** | Adherence to existing codebase conventions, naming, patterns, style |
| **Testing** | Test coverage, test quality, edge cases, appropriate test level |
| **Readability** | Code clarity, naming, complexity, documentation |
| **Error Handling** | Result pattern, fail-fast, edge cases, logging, resilience |

**Additionally, select 3-6 context-specific categories** that are relevant to the particular PR. Examples:

- **Concurrency** — for code with shared state or database writes
- **API Design** — for new/changed endpoints (REST conventions, status codes, OpenAPI)
- **Database** — for migrations, queries, indexes, EF Core usage
- **Frontend** — for Lit components, Shadow DOM, accessibility
- **Breaking Changes** — when public API surface is affected
- **Observability** — for logging, tracing, health checks
- **Configuration** — for DI registration, options, environment handling
- **Domain Modeling** — for value objects, aggregates, invariants

### Output Format

**1. Score table** at the top:

```markdown
| Category | Score | Comment |
|----------|------:|---------|
| **Overall** | **75** | |
| Security | 90 | No issues found |
| Performance | 60 | Potential N+1 in GetUsers |
| Architecture | 80 | Clean separation |
| ... | ... | ... |
```

**2. Categorized comments** below the table, grouped by severity:

- **Critical** — Must fix before merge (security vulnerabilities, data loss risks, broken functionality)
- **Important** — Should fix, significant quality concern (performance issues, missing tests, architectural violations)
- **Suggestion** — Nice to have, improvement opportunity (readability, minor refactoring, alternative approaches)
- **Praise** — Highlight good patterns and decisions worth noting

Each comment should reference the specific file and line, explain the issue, and suggest a fix when applicable.

## Umbraco Brand Guidelines

When creating UI, marketing materials, or any visual assets, follow the official Umbraco brand guidelines.

### Brand Colors

| Name | HEX | RGB | Usage |
|------|-----|-----|-------|
| **Umbraco Blue** (primary) | `#283a97` | 40, 58, 148 | Primary brand color |
| **Umbraco Pink** | `#f5c1bc` | 245, 193, 188 | Accent color |
| **Umbraco Dark Blue** | `#1b264f` | 27, 38, 79 | Dark backgrounds, text |
| **Umbraco Cream** | `#fff0ea` | 255, 240, 234 | Light backgrounds |
| **White** | `#ffffff` | 255, 255, 255 | Backgrounds |
| **Black** | `#000000` | 0, 0, 0 | Text |

### Typography

- **Font Family:** Lato (exclusively)
- Use Lato for all text in Umbraco products and materials
- Download: https://drive.google.com/file/d/1CSlVBlG8fKnXtzmxYihiHeZgSw9h_U9c/view

### Icons

- **Icon Library:** Lucide (open source)
- **Browse icons:** https://lucide.dev/icons/
- All icons in Umbraco products must come from Lucide
- Follow Brand Guidelines for proper icon usage and sizing

### Logo Usage

Four logo variants exist:
- **Horizontal Logo** - Primary usage
- **Vertical Logo** - When horizontal space is limited
- **Logotype** - Text only
- **Logomark** - Icon only

Refer to the official Brand Guidelines document for Do's and Don'ts.

### Resources

- **Brand Guidelines PDF:** https://drive.google.com/file/d/1t9V-hq7d6PEl1yfKX-U4YsKwYQwi6WCL/view
- **Design Hub:** https://sites.google.com/umbraco.dk/designhub/brand (requires Umbraco login)
- **Image Gallery:** https://drive.google.com/drive/folders/1Rvlowi2lgacTkEvjUDnOD-TAiSHJ64gr
- **Design Requests:** Contact Annie (ami@umbraco.dk) or use Airtable Request Form: https://airtable.com/appGsT7Q6pDXZnpKd/shrQbH7Qpbv9lLLJr
