# Reference Architecture — Job Tracker Application

**Version:** 1.0  
**Date:** 2026-05-12  
**Author:** Mark Riggs  
**Status:** Current

---

## Overview

The Job Tracker Application is a cloud-native, event-driven distributed system built on microservices principles. It demonstrates enterprise architecture patterns including API gateway, async messaging, real-time push, federated identity, blob storage, AI integration, and infrastructure as code — all deployed to production on a Linux VPS via Docker Compose, with Kubernetes as the target orchestration platform.

The architecture is documented at two C4 levels: **System Context** (the system and its external dependencies) and **Container** (the internal building blocks and how they communicate).

---

## Level 1 — System Context

```mermaid
C4Context
    title System Context — Job Tracker Application

    Person(user, "Job Seeker", "Tracks job applications, contacts, and generates tailored resumes during an active job search")

    System(jobTracker, "Job Tracker Application", "Distributed microservices system — manages job pipeline, contacts, journals, resumes, and AI-assisted resume generation")

    System_Ext(auth0, "Auth0", "Federated identity provider — issues JWTs, manages user sessions and refresh tokens via OIDC/OAuth2")
    System_Ext(anthropic, "Anthropic Claude API", "Large language model — generates tailored resume content from job descriptions and experience profiles")
    System_Ext(sendgrid, "SendGrid / SMTP", "Email relay — delivers user feedback notifications to the configured recipient")
    System_Ext(duckdns, "DuckDNS", "Dynamic DNS — resolves production domain to VPS IP address")
    System_Ext(githubpages, "GitHub Pages", "CDN / static hosting — serves the React SPA; CI/CD via GitHub Actions on push to main")
    System_Ext(akamai, "Akamai VPS", "Cloud virtual machine (Debian 12) — hosts all containerised backend services via Docker Compose; provisioned via Terraform")

    Rel(user, jobTracker, "Uses", "HTTPS / Browser")
    Rel(jobTracker, auth0, "Delegates authentication", "HTTPS / OIDC")
    Rel(jobTracker, anthropic, "Assembles resume prompts via", "HTTPS / REST")
    Rel(jobTracker, sendgrid, "Delivers email notifications via", "SMTP / TLS port 587")
    Rel(jobTracker, duckdns, "Domain resolved via", "DNS")
    Rel(jobTracker, githubpages, "React frontend hosted on", "HTTPS")
    Rel(jobTracker, akamai, "Backend services deployed on", "SSH / Docker Compose")
```

---

## Level 2 — Container Diagram

```mermaid
C4Container
    title Container Diagram — Job Tracker Application

    Person(user, "Job Seeker", "")

    System_Ext(auth0, "Auth0", "Identity / JWT issuer")
    System_Ext(anthropic, "Anthropic Claude API", "AI / LLM")
    System_Ext(smtp, "SendGrid SMTP", "Email delivery")

    Container_Boundary(cdn, "GitHub Pages (CDN)") {
        Container(react, "React SPA", "TypeScript · Vite · TanStack Query · SignalR client", "Job pipeline UI — dashboard, job detail, contacts, journal, resume, AI prompt generation")
    }

    Container_Boundary(vps, "Akamai VPS — Docker Compose") {

        Container(npm, "Nginx Proxy Manager", "Nginx · Let's Encrypt", "TLS termination — routes HTTPS traffic to the API gateway on the internal Docker network")
        Container(gateway, "API Gateway", ".NET 8 · YARP Reverse Proxy", "Single entry point — Auth0 JWT validation, WebSocket upgrade, routes to downstream services")

        Container(jobsvc, "Job Service", ".NET 8 · REST API", "Accepts job create/update — publishes full payload to Kafka and returns HTTP 202 immediately. No direct DB write.")
        Container(jobconsumer, "Job Service Consumer", ".NET 8 · BackgroundService · SignalR Hub", "Consumes job.application.* from Kafka — writes to PostgreSQL, pushes jobCreated/jobUpdated events to connected React clients via SignalR")
        Container(contactsvc, "Contact Service", ".NET 8 · EF Core", "Contact management — supports many-to-many contact/job linking with role types")
        Container(journalsvc, "Journal Service", ".NET 8 · EF Core", "Activity journal — interaction types, notes, and dates per job requisition")
        Container(resumesvc, "Resume Service", ".NET 8 · MinIO SDK", "Application document management — upload, download, delete, and link one resume and one cover letter per job application")
        Container(expsvc, "Experience Service", ".NET 8 · MinIO SDK", "Work experience document store — feeds content into AI resume prompt assembly")
        Container(aisvc, "AI Service", ".NET 8 · Anthropic SDK", "AI profile CRUD and server-side prompt assembly — fetches job description and experience content, returns assembled Claude.ai prompt")
        Container(notificationsvc, "Notification Service", ".NET 8 · BackgroundService · MailKit", "Dual Kafka consumer — consumes job events (for future notifications) and feedback.submitted (delivers email via SMTP)")

        ContainerDb(postgres, "PostgreSQL 16", "Relational database", "Primary data store — each service owns its own schema; EF Core migrations auto-applied on startup")
        ContainerDb(kafka, "Apache Kafka 7.6", "Event streaming platform", "Async message bus — topics: job.application.created, job.application.updated, feedback.submitted")
        ContainerDb(minio, "MinIO", "S3-compatible object store", "Blob storage — resumes, cover letters, experience documents, AI-generated resume output")
    }

    Rel(user, react, "Uses", "HTTPS")
    Rel(react, npm, "All API calls + SignalR WebSocket", "HTTPS / WSS")
    Rel(npm, gateway, "Proxies to", "HTTP / WS — internal Docker network")

    Rel(gateway, auth0, "Validates JWT with", "HTTPS / JWKS")
    Rel(gateway, jobsvc, "Routes /api/jobs (write)", "HTTP")
    Rel(gateway, jobconsumer, "Routes /hubs/jobs (SignalR)", "HTTP / WS — LongPolling")
    Rel(gateway, contactsvc, "Routes /api/contacts", "HTTP")
    Rel(gateway, journalsvc, "Routes /api/jobs/{id}/journal", "HTTP")
    Rel(gateway, resumesvc, "Routes /api/resumes, /api/jobs/{id}/documents", "HTTP")
    Rel(gateway, expsvc, "Routes /api/experience-profiles", "HTTP")
    Rel(gateway, aisvc, "Routes /api/ai-profiles, /api/jobs/{id}/generated-resumes", "HTTP")
    Rel(gateway, notificationsvc, "Routes /api/feedback", "HTTP")

    Rel(jobsvc, kafka, "Publishes job.application.created / updated", "Kafka producer")
    Rel(notificationsvc, kafka, "Publishes feedback.submitted", "Kafka producer")
    Rel(jobconsumer, kafka, "Consumes job.application.*", "Kafka consumer group")
    Rel(notificationsvc, kafka, "Consumes job.application.*, feedback.submitted", "Kafka consumer group")

    Rel(jobconsumer, postgres, "Writes job records", "EF Core / TCP")
    Rel(contactsvc, postgres, "Reads / writes contacts", "EF Core / TCP")
    Rel(journalsvc, postgres, "Reads / writes journal entries", "EF Core / TCP")
    Rel(aisvc, postgres, "Reads / writes AI profiles", "EF Core / TCP")

    Rel(resumesvc, minio, "Stores and retrieves resume and cover letter files", "S3 API / HTTP")
    Rel(expsvc, minio, "Stores and retrieves experience documents", "S3 API / HTTP")
    Rel(aisvc, minio, "Stores AI-generated resume output", "S3 API / HTTP")

    Rel(aisvc, anthropic, "Assembles prompts via", "HTTPS / REST")
    Rel(notificationsvc, smtp, "Delivers email via", "SMTP / TLS 587")

    Rel(jobconsumer, react, "Pushes real-time job events to", "SignalR — LongPolling transport")
```

---

## Key Architectural Decisions

| Concern | Decision | Rationale |
|---|---|---|
| Service decomposition | Domain-per-service (polyrepo) | Independent deployability, clear ownership boundaries |
| Write path for jobs | Kafka publish → 202 Accepted, no synchronous DB write | Decouples HTTP response time from persistence; publisher and consumer independently scalable |
| Status updates | Synchronous REST (no Kafka) | Lightweight field update; async overhead not justified |
| Real-time push | SignalR via Job Service Consumer | Consumer owns the DB write and is best placed to signal completion |
| SignalR transport | LongPolling (not WebSocket) | WebSocket frames do not traverse the NPM → YARP proxy chain reliably in this topology |
| Identity | Auth0 (external IdP) | Eliminates credential management, supports OIDC/OAuth2, refresh token rotation |
| Blob storage | MinIO (S3-compatible) | Self-hosted, S3 API compatible — zero vendor lock-in, drop-in replacement for AWS S3 |
| Database strategy | PostgreSQL, schema-per-service | Single engine, isolated schemas; migrations owned by each service |
| IaC | Terraform (Akamai/Linode provider) | VM, SSH key, and firewall declared as code; reproducible provisioning |
| Frontend deployment | GitHub Pages + GitHub Actions | Zero-cost static hosting; CI gates test pass before build |
| AI integration | User-triggered, Claude.ai model first | Server assembles the prompt; user pastes into Claude.ai — no API cost until API model is enabled |

Full decision records: [Architecture Decision Records](../adr/)

---

## Infrastructure Topology

```
Internet
  └── DuckDNS (dynamic DNS → VPS IP)
        └── Akamai VPS :443 / :80
              └── Nginx Proxy Manager (TLS termination, Let's Encrypt)
                    └── API Gateway :8080 (YARP, Auth0 JWT validation)
                          ├── Job Service :8080
                          ├── Job Service Consumer :8080 (+ SignalR hub)
                          ├── Contact Service :8080
                          ├── Journal Service :8080
                          ├── Resume Service :8080
                          ├── Experience Service :8080
                          ├── AI Service :8080
                          └── Notification Service :8080
                                └── [Kafka, PostgreSQL, MinIO — internal Docker network]

GitHub Pages (CDN)
  └── React SPA → all API calls → DuckDNS domain → Nginx → Gateway → services
```

---

## Repository Structure

14 repositories under a polyrepo strategy — each service, the gateway, infrastructure config, shared libraries, and documentation maintained independently.

| Repository | Role |
|---|---|
| job-tracker-app-gateway | YARP API gateway |
| job-tracker-app-job-service | Job domain — Kafka publisher |
| job-tracker-app-job-service-consumer | Kafka consumer, DB writer, SignalR hub |
| job-tracker-app-contact-service | Contact domain |
| job-tracker-app-journal-service | Activity journal domain |
| job-tracker-app-resume-service | Application document management (resumes and cover letters) |
| job-tracker-app-experience-service | Experience document storage |
| job-tracker-app-ai-service | AI profile management, prompt assembly |
| job-tracker-app-notification-service | Notification and email delivery |
| job-tracker-app-user-service | User profile domain (planned) |
| job-tracker-app-web-react | React frontend |
| job-tracker-app-web-angular | Angular frontend (Phase 2) |
| job-tracker-app-infrastructure | Docker Compose, Terraform, Nginx config |
| job-tracker-app-docs | Architecture docs, ADRs, guides |

---

## EA Artifact Index

| Document | Purpose |
|---|---|
| [Executive Architecture Brief](executive-architecture-brief.md) | One-page strategic summary for non-technical stakeholders |
| [Technology Roadmap](technology-roadmap.md) | Phase 1/2/3 capability plan with rationale and governance principles |
| [Engineering Standards & Governance](engineering-standards-governance.md) | Coding standards, API design, ADR process, testing requirements |
| [Security Architecture](security-architecture.md) | Threat model, identity flows, network topology, data classification |
| [Architecture Decision Records](../adr/) | Full decision history — 10 ADRs covering all major technology choices |
