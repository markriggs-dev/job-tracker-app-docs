# Engineering Standards & Governance Framework — Job Tracker Application

**Version:** 1.0  
**Date:** 2026-05-12  
**Author:** Mark Riggs  
**Status:** Current

---

## Purpose

This document defines the engineering standards, development practices, and governance processes that apply across all services in the Job Tracker Application. It exists to ensure consistency, quality, and independent deployability across a polyrepo microservices architecture.

These standards are enforced through CI pipelines, code review, and the Architecture Decision Record (ADR) process.

---

## 1. Branching & Release Strategy

| Rule | Detail |
|---|---|
| Primary branches | `main` (production) and `develop` (integration) |
| Feature work | Branch from `develop`; PR back to `develop` |
| Release | Merge `develop` → `main` triggers production deployment |
| Direct commits to `main` | Prohibited |
| Commit style | Conventional Commits (`feat:`, `fix:`, `ci:`, `docs:`, `refactor:`) |

Merges to `main` require passing CI (test gate + build gate). No exceptions.

---

## 2. API Design Standards

### REST Conventions

| Convention | Standard |
|---|---|
| Resource naming | Plural nouns: `/api/jobs`, `/api/contacts` |
| Identifiers | UUIDs in path: `/api/jobs/{id}` |
| HTTP verbs | GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE |
| Success codes | 200 (OK), 201 (Created), 202 (Accepted — async), 204 (No Content) |
| Error codes | 400 (validation), 401 (unauthenticated), 403 (forbidden), 404 (not found), 500 (server error) |
| Response envelope | No wrapper — return resource directly or array |
| Date format | ISO 8601 / RFC 3339 (`DateTimeOffset` in .NET) |

### Async Write Pattern

Operations that trigger downstream processing (Kafka publish) return **HTTP 202 Accepted** immediately. The response body contains the resource ID. Clients receive the completed state via SignalR push event.

### Versioning

API versioning is deferred to Phase 2. When introduced, URL-based versioning (`/api/v2/`) will be the standard.

---

## 3. Coding Standards

### C# / .NET

| Rule | Standard |
|---|---|
| Target framework | `net8.0` across all services |
| Nullable reference types | Enabled on all projects |
| Async | `async`/`await` throughout; no `.Result` or `.Wait()` |
| Dependency injection | Constructor injection only; no service locator |
| Repository pattern | All data access behind repository interfaces |
| DTOs | Separate request/response records; never expose EF entities directly |
| Logging | `ILogger<T>` injected; structured logging only (no string interpolation in log calls) |
| Configuration | `IOptions<T>` for typed config; never read `IConfiguration` directly in services |

### TypeScript / React

| Rule | Standard |
|---|---|
| Strict mode | `strict: true` in `tsconfig.json` |
| Interfaces | `import type` for interface imports (Vite 6 requirement) |
| Enums | `const` objects with `as const`; no TypeScript enums (Vite 6) |
| State management | TanStack Query for server state; React `useState` for local UI state |
| API calls | Axios instance via `useAxiosWithAuth` hook; all calls through gateway |
| Styling | CSS Modules; no inline styles |
| Component size | Single responsibility; extract to sub-components when JSX exceeds ~100 lines |

---

## 4. Testing Requirements

| Tier | Framework | Requirement |
|---|---|---|
| Unit tests (.NET) | xUnit + Moq + FluentAssertions | Required for all service-layer logic; CI gated |
| Unit tests (React) | Vitest | Required for utility functions and hooks; CI gated |
| Integration tests | Planned (Phase 2) | EF Core + PostgreSQL via testcontainers; Kafka consumer flows |
| E2E tests | Not planned | Out of scope for portfolio phase |

**CI gate:** All merges to `main` and `develop` must pass the test suite. A failing test blocks the merge.

---

## 5. Security Standards

| Standard | Requirement |
|---|---|
| Authentication | All API endpoints secured via Auth0 JWT validated at the API Gateway |
| Authorization | `userId` claim extracted from validated JWT; all data queries scoped to `userId` |
| Secrets | Never committed to source control; stored in `.env` (gitignored); `.env.example` documents required keys |
| TLS | All production traffic over HTTPS/WSS; Let's Encrypt certificates via Nginx Proxy Manager |
| CORS | Configured at gateway; allowed origins declared via environment variable |
| Service-to-service | No direct service-to-service calls without bearer token forwarding |

See [Security Architecture](security-architecture.md) for the full threat model.

---

## 6. Documentation Requirements

| Artifact | Owner | When Required |
|---|---|---|
| ADR | Architect | Any significant technology or design decision |
| README | Service owner | Every repository; includes purpose, local run, and environment variables |
| `.env.example` | Service owner | Every service with external dependencies |
| API contract | Service owner | Swagger/OpenAPI auto-generated via Swashbuckle |
| Architecture diagrams | Architect | Updated when topology changes |

---

## 7. Architecture Decision Record (ADR) Process

An ADR is required for any decision that:

- Introduces a new technology or framework
- Changes a cross-service communication pattern
- Affects security, data storage, or deployment topology
- Represents a deliberate tradeoff between two viable options

### ADR Lifecycle

```
Proposed → Accepted → Superseded (if replaced by a newer decision)
```

### ADR Template

```markdown
# ADR-NNN — [Title]
**Status:** Proposed | Accepted | Superseded  
**Date:** YYYY-MM-DD

## Context
What problem are we solving and why does it matter?

## Decision
What did we decide?

## Consequences
What are the tradeoffs? What becomes easier and what becomes harder?
```

All ADRs are stored in `job-tracker-app-docs/adr/` and indexed in the [Technology Roadmap](technology-roadmap.md).

---

## 8. Dependency & Package Management

| Rule | Detail |
|---|---|
| .NET packages | NuGet; versions pinned in `.csproj`; no floating versions (`*`) |
| npm packages | Exact versions in `package-lock.json`; committed to source control |
| Package updates | Deliberate; reviewed for breaking changes; accompanied by a test run |
| New dependencies | Require justification; prefer packages with active maintenance and broad adoption |

---

## 9. Code Review Standards

| Standard | Requirement |
|---|---|
| PR size | Prefer small, focused PRs; one concern per PR |
| Self-review | Author reviews diff before opening PR |
| CI required | PR cannot merge with failing CI |
| Review focus | Correctness, security, test coverage, adherence to standards in this document |
| Merge strategy | Squash for feature branches; merge commit for release merges (`develop` → `main`) |

---

## Related Documents

- [Reference Architecture](reference-architecture.md)
- [Technology Roadmap](technology-roadmap.md)
- [Security Architecture](security-architecture.md)
- [ADR Index](../adr/)
