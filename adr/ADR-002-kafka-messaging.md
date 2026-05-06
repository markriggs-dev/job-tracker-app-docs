# ADR-002: Kafka as an async work queue for job application events

**Status:** Accepted  
**Date:** 2026-04-11  
**Author:** Mark  
**Last updated:** 2026-05-05

## Context

Services need to communicate with each other without tight coupling. The job service handles create and edit operations for job requisitions. Rather than processing all downstream work synchronously in the request path, a decision is required on how to offload that work so the API can respond immediately and consumers process independently.

## Decision

Apache Kafka will be used as an async work queue. The job service publishes a full job application payload to Kafka on create (`job.application.created`) and edit (`job.application.updated`) and returns **202 Accepted** immediately — it does not write to the database. The Job Service Consumer subscribes to both topics, writes the record to the database via EF Core, and pushes a real-time SignalR event to the React client so it can invalidate its local query cache.

Status changes are handled synchronously — they are lightweight field updates that do not require downstream processing and remain in the Job Service with a direct DB write.

AI analysis is never triggered automatically. It is user-initiated via an explicit action on the job detail page, so AI-related events are not part of the Kafka pipeline.

## Rationale

- The job service does not need to know that any consumer exists. Producers and consumers are fully decoupled.
- Publishing the full job payload (all database fields) to Kafka means any consumer has everything it needs without calling back to the job service.
- Kafka topics act as a durable event log, enabling event replay and audit history.
- New consumers can be added to existing topics without modifying the producer, supporting future extensibility. The AI service, for example, can subscribe to `job.application.created` to pre-process job descriptions without any changes to the job service.
- Under high load, consumers process at their own pace without the API blocking or dropping requests.
- Kafka is the industry standard for distributed event streaming and demonstrates enterprise-grade messaging architecture.
- SignalR push from the consumer closes the loop to the React client — the UI refreshes when the DB write completes rather than relying on polling or optimistic updates.

## Alternatives considered

- **REST-based synchronous calls:** Introduces tight coupling between services and cascading failure risk.
- **RabbitMQ:** A solid alternative message broker, but Kafka's log-based model and scalability story are stronger for portfolio demonstration.
- **In-process events:** Only viable within a monolith, not applicable here.
- **Polling for DB write completion:** Simple but chatty and adds latency. SignalR push is more efficient and demonstrates real-time capability.

## Consequences

- Requires a running Kafka instance in the local development environment via Docker Compose.
- Job Service Consumer is a separate service that owns the DB write for create/edit — the Job Service is publisher-only for these operations.
- Job Service Consumer hosts the SignalR hub; the API Gateway proxies WebSocket connections to it.
- Teams need familiarity with Kafka consumer group semantics.
- Enables future features like event sourcing and audit logging with minimal additional work.

## Kafka topics

| Topic | Publisher | Trigger | Payload |
|-------|-----------|---------|---------|
| `job.application.created` | Job Service | POST /api/jobs | JobReqId, UserId, UserEmail, CompanyName, RoleTitle, SourceUrl, CompanyCareerPortalUrl, JobDescription, DateDiscovered, ApplicationExpiryDate, OccurredAt |
| `job.application.updated` | Job Service | PUT /api/jobs/{id} | JobReqId, UserId, UserEmail, CompanyName, RoleTitle, SourceUrl, CompanyCareerPortalUrl, JobDescription, DateDiscovered, ApplicationExpiryDate, DateSubmitted, OccurredAt |

## Current consumers

| Consumer | Topic | Action |
|----------|-------|--------|
| Job Service Consumer | `job.application.created` | Writes new job requisition to DB; pushes `jobCreated` SignalR event |
| Job Service Consumer | `job.application.updated` | Updates job requisition in DB; pushes `jobUpdated` SignalR event |
| Notification Service | `job.application.created` | Logs event (email notifications reserved for future use cases) |
| Notification Service | `job.application.updated` | Logs event |
