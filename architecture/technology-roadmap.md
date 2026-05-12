# Technology Roadmap — Job Tracker Application

**Version:** 1.0  
**Date:** 2026-05-12  
**Author:** Mark Riggs  
**Status:** Current

---

## Summary

The Job Tracker Application is built in three phases, each building on the previous layer: **Foundation** establishes the distributed core; **Operational Excellence** adds the instrumentation, automation, and multi-framework support that enterprise systems require; **Scale & Extend** moves to full container orchestration and broadens the platform's capability surface.

---

## Phase 1 — Foundation ✅ Complete

**Objective:** Build a production-grade distributed system that solves a real problem, deployed to cloud infrastructure, with CI/CD and a working AI integration.

### Delivered

| Capability | Technology | Notes |
|---|---|---|
| Microservices architecture | .NET 8 / C# (9 services) | Domain-per-service, polyrepo, independently deployable |
| API Gateway | YARP Reverse Proxy | Single entry point, JWT validation, WebSocket support |
| Async event streaming | Apache Kafka | job.application.created/updated, feedback.submitted topics |
| 202 async write pattern | Job Service → Kafka → Consumer | Publisher returns immediately; consumer owns DB write |
| Real-time push | SignalR (LongPolling transport) | jobCreated/jobUpdated events invalidate TanStack Query cache |
| Federated identity | Auth0 / OIDC / OAuth2 | JWT validation at gateway; refresh token rotation; 401 retry interceptor |
| Relational database | PostgreSQL 16 / EF Core | Schema-per-service; auto-migrate on startup |
| Blob storage | MinIO (S3-compatible) | Resumes, cover letters, experience documents, AI-generated output |
| Email delivery | MailKit / SendGrid SMTP | Feedback submission notification; Kafka-decoupled send |
| AI integration | Anthropic Claude / Anthropic SDK | Server-assembled prompts; Claude.ai model (paste flow) + API model (planned) |
| React frontend | React 19 / TypeScript / Vite | Dashboard, job detail, contacts, journal, documents (resume + cover letter), AI tabs; Auth0 SPA |
| TLS & reverse proxy | Nginx Proxy Manager / Let's Encrypt | Production TLS termination |
| Infrastructure as code | Terraform (Akamai/Linode provider) | VM, SSH key, firewall declared as code |
| Container runtime | Docker / Docker Compose | All services containerised; 2-stage Dockerfiles |
| CI pipeline | GitHub Actions | Test gate → build gate on every push to main/develop |
| Frontend deployment | GitHub Pages + GitHub Actions | SPA with 404 redirect pattern; Vite base URL conditional |
| Unit testing | xUnit + Moq (17 tests) / Vitest (13 tests) | Service layer tests; utility tests; CI gated |
| Architecture governance | 10 ADRs | Kafka, gateway, SignalR transport, testing strategy, AI integration, Terraform |

---

## Phase 2 — Operational Excellence 🔲 Planned

**Objective:** Harden the platform with observability, deployment automation, expanded test coverage, and a second frontend framework to demonstrate multi-stack capability.

### Planned Capabilities

| Capability | Technology | Rationale |
|---|---|---|
| Centralized structured logging | Seq (single Docker container) | Native .NET/Serilog integration; query logs across all services in one UI |
| Distributed tracing | OpenTelemetry | Correlate requests across service boundaries; identify latency hot spots |
| Metrics & dashboards | Prometheus + Grafana | Infrastructure and application metrics; alerting |
| CI/CD to VPS | GitHub Actions + SSH deploy | Eliminate manual `git pull` + `docker compose up` after every merge to main |
| Integration tests | EF Core + PostgreSQL (testcontainers) | Test Kafka consumer flows and DB writes end-to-end; Docker-in-Docker in CI |
| User Service | .NET 8 / EF Core | User profile management — preferences, settings, auth metadata |
| Angular frontend | Angular 19 / TypeScript | Parallel frontend demonstrating multi-framework strategy; shares same API gateway |

---

## Phase 3 — Scale & Extend 🔲 Future

**Objective:** Graduate the platform to production-grade Kubernetes orchestration, extend AI capabilities, and expand the surface area of the application.

### Planned Capabilities

| Capability | Technology | Rationale |
|---|---|---|
| Container orchestration | Kubernetes (AKS or EKS) | Horizontal pod autoscaling, rolling deployments, health probes, self-healing |
| API-based AI generation | Anthropic API (direct) | In-app resume generation without leaving the platform; generated files stored in MinIO |
| Service mesh | Istio or Linkerd | mTLS between services, traffic management, circuit breaking at infrastructure level |
| Secrets management | HashiCorp Vault or Azure Key Vault | Remove credentials from `.env` files; dynamic secret rotation |
| Multi-region deployment | Cloud provider regions | Active-passive or active-active for resilience and latency |
| Async job status | Webhook / polling pattern | Allow long-running AI jobs to report completion asynchronously |
| Mobile application | React Native | Extend job tracking to mobile — reuses existing API gateway and services |

---

## Technology Governance Principles

These principles guide technology selection across all phases and serve as the standing evaluation criteria for any new addition to the stack.

1. **Deliberate selection** — Every technology must have a documented rationale (ADR). No framework is added because it is trending.
2. **Operational simplicity first** — Prefer managed services and well-supported runtimes. Add complexity only when the tradeoff is clearly justified.
3. **Independent deployability** — Each service must be buildable, testable, and deployable without knowledge of other services.
4. **Test before ship** — All merges to `main` pass a CI test gate. Integration tests are required for any Kafka or database-touching code path.
5. **Infrastructure as code** — All cloud resources are declared in Terraform. No manual provisioning in production.
6. **Security at the boundary** — Auth enforced at the gateway; all services trust the validated JWT. No service-to-service trust without the token.
7. **Observability by design** — Logging, tracing, and metrics are first-class citizens, not retrofitted features.

---

## ADR Index

| ADR | Decision |
|---|---|
| [ADR-001](../adr/ADR-001-microservices-architecture.md) | Microservices with Docker and Kubernetes |
| [ADR-002](../adr/ADR-002-kafka-messaging.md) | Kafka as async message bus |
| [ADR-003](../adr/ADR-003-auth0-identity.md) | Auth0 for federated identity |
| [ADR-004](../adr/ADR-004-postgresql.md) | PostgreSQL as primary data store |
| [ADR-005](../adr/ADR-005-blob-storage.md) | MinIO for blob storage |
| [ADR-006](../adr/ADR-006-frontend-frameworks.md) | React (Phase 1) and Angular (Phase 2) |
| [ADR-007](../adr/ADR-007-signalr-transport.md) | SignalR LongPolling transport |
| [ADR-008](../adr/ADR-008-ai-integration.md) | AI integration via Anthropic Claude |
| [ADR-009](../adr/ADR-009-terraform-vps-provisioning.md) | Terraform for VPS provisioning |
| [ADR-010](../adr/ADR-010-testing-strategy.md) | xUnit + Vitest testing strategy |
