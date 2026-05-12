# Executive Architecture Brief — Job Tracker Application

**Version:** 1.0  
**Date:** 2026-05-12  
**Author:** Mark Riggs  
**Audience:** Non-technical stakeholders, engineering leadership, hiring managers

---

## What This System Does

The Job Tracker Application is a cloud-native platform that helps job seekers manage their entire job search in one place — tracking applications, contacts, activity journals, uploaded resumes and cover letters, and AI-assisted resume generation tailored to each job description.

It is a fully deployed, production system solving a real problem, built on enterprise distributed systems patterns.

---

## Why It Was Built This Way

The system was deliberately designed to enterprise standards relative to its feature scope. The goal was not the simplest way to track job applications — it was to demonstrate the architectural patterns, governance practices, and engineering disciplines that large-scale systems require.

Every technology choice is documented in an Architecture Decision Record (ADR). Nothing was added because it was trending. Nothing was omitted to save time.

---

## Architecture at a Glance

| Layer | Technology | Role |
|---|---|---|
| Frontend | React 19 / TypeScript / GitHub Pages | SPA served from CDN; zero infrastructure cost |
| Identity | Auth0 / OIDC / OAuth2 | Federated identity — no credential storage in the application |
| API Gateway | YARP / .NET 8 | Single entry point — JWT validation, routing, WebSocket support |
| Services (×8) | .NET 8 microservices | Domain-isolated, independently deployable, polyrepo |
| Async messaging | Apache Kafka | Decoupled write path; publisher and consumer scale independently |
| Real-time push | SignalR | Live UI updates on job create/edit without client polling |
| Database | PostgreSQL 16 / EF Core | Schema-per-service ownership; auto-migrate on startup |
| Blob storage | MinIO (S3-compatible) | Resumes, cover letters, experience documents, AI-generated output |
| AI integration | Anthropic Claude | Server-assembled prompts; tailored resume generation |
| Infrastructure | Docker Compose / Terraform | Fully containerised; infrastructure declared as code |
| Hosting | Akamai VPS + GitHub Pages | Production-deployed; TLS via Let's Encrypt |

---

## Key Strategic Decisions

**Microservices over a monolith** — Each domain (jobs, contacts, journal, resumes, AI) is an independent service with its own schema, deployment pipeline, and team ownership boundary. The cost is operational complexity; the benefit is independent scalability and clear accountability.

**Async write path via Kafka** — Job creation publishes to Kafka and returns HTTP 202 Accepted immediately. A separate consumer writes to the database and notifies the client via SignalR. This decouples HTTP response time from persistence latency and enables publisher and consumer to scale independently — the same pattern used in high-throughput financial and e-commerce systems.

**Federated identity via Auth0** — The application never stores credentials or manages sessions. Auth0 handles MFA, refresh token rotation, and brute-force protection. This eliminates an entire class of security risk and reduces compliance surface area.

**Infrastructure as code via Terraform** — The VPS, SSH key, and firewall are declared in Terraform and stored in source control. The environment is reproducible from scratch in under 10 minutes. No manual provisioning steps exist.

**S3-compatible blob storage via MinIO** — MinIO provides the S3 API without vendor lock-in. Migrating to AWS S3 or Azure Blob Storage in Phase 3 requires changing an endpoint and credentials, not rewriting storage code. This is the same portability principle that drives the MinIO adoption in enterprises moving toward multi-cloud.

**CI/CD with test gates** — Every merge to `main` and `develop` passes a test suite before a build is produced. The pipeline is the enforcer of quality standards — not individual discipline.

---

## Risk Profile

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Single VPS is a single point of failure | Medium | High | Accepted for portfolio phase; Kubernetes multi-node target in Phase 3 |
| Secrets in `.env` files | Low | High | `.env` gitignored; Vault / Key Vault planned for Phase 3 |
| No observability tooling | Certain | Medium | Seq + OpenTelemetry + Prometheus/Grafana planned for Phase 2 |
| Manual VPS deployment | Certain | Low | GitHub Actions SSH deploy planned for Phase 2 |
| No server-side encryption at rest | Low | Medium | Host-level encryption present; application-level planned for Phase 3 |

---

## Value Delivered

| Capability | Enterprise Pattern |
|---|---|
| 8 independently deployable services | Microservices / domain decomposition |
| Kafka async write + 202 Accepted | Event-driven architecture |
| SignalR real-time push | Push notifications without polling |
| Auth0 OIDC + JWT gateway validation | Zero-trust identity at the perimeter |
| MinIO S3-compatible blob storage | Vendor-neutral cloud storage abstraction |
| Terraform VPS provisioning | Infrastructure as code / reproducible environments |
| GitHub Actions CI/CD with test gates | DevOps / shift-left quality |
| 10 Architecture Decision Records | Architecture governance and decision traceability |
| Engineering Standards & Governance framework | Team operating model and quality standards |
| Phase 2 / Phase 3 technology roadmap | Strategic planning and investment sequencing |

---

## Investment Summary

**Phase 1 — Foundation** ✅ Complete  
Production-grade distributed system deployed to cloud infrastructure. Eight microservices, event streaming, real-time push, federated identity, blob storage, AI integration, CI/CD, and IaC — all running in production.

**Phase 2 — Operational Excellence** 🔲 Planned  
Observability (Seq, OpenTelemetry, Grafana), automated deployment, integration test coverage, Angular frontend, and User Service.

**Phase 3 — Scale & Extend** 🔲 Future  
Kubernetes orchestration, service mesh, secrets management, multi-region deployment, and mobile application.

---

## Further Reading

| Document | Purpose |
|---|---|
| [Reference Architecture](reference-architecture.md) | C4 System Context and Container diagrams |
| [Technology Roadmap](technology-roadmap.md) | Phase 1/2/3 capability plan with rationale |
| [Engineering Standards & Governance](engineering-standards-governance.md) | Coding standards, API design, ADR process, testing requirements |
| [Security Architecture](security-architecture.md) | Threat model, identity flows, network topology, data classification |
| [Architecture Decision Records](../adr/) | Full decision history — 10 ADRs covering all major technology choices |
