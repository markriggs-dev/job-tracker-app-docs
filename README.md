# job-tracker-app-docs

Documentation repository for the Job Tracker application. Contains architecture decision records, user stories, system diagrams, and developer guides.

## Contents

| Folder | Contents |
|--------|----------|
| `adr/` | Architecture Decision Records |
| `user-stories/` | User stories by domain |
| `diagrams/` | System architecture and data flow diagrams |
| `guides/` | Developer setup and contribution guides |
| `api/` | API contract documentation |

## Documents

### Architecture Decision Records
- [ADR-001: Microservices with Docker and Kubernetes](adr/ADR-001-microservices-architecture.md)
- [ADR-002: Kafka as inter-service message bus](adr/ADR-002-kafka-messaging.md)
- [ADR-003: Auth0 for identity management](adr/ADR-003-auth0-identity.md)
- [ADR-004: PostgreSQL as primary data store](adr/ADR-004-postgresql.md)
- [ADR-005: Blob storage for resume files](adr/ADR-005-blob-storage.md)
- [ADR-006: React and Angular frontends](adr/ADR-006-frontend-frameworks.md)

### Guides
- [Local development setup](guides/local-development.md)
- [Repository overview](guides/repository-overview.md)
- [Contributing](guides/contributing.md)
