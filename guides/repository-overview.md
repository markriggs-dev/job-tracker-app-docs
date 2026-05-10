# Repository overview

The Job Tracker application uses a polyrepo structure. Each service and frontend lives in its own repository, with a shared infrastructure repository for local development and deployment configuration.

## Repository map

| Repository | Type | Description |
|------------|------|-------------|
| job-tracker-app-gateway | .NET 8 | API gateway, routing, JWT validation |
| job-tracker-app-job-service | .NET 8 | Job requisition domain |
| job-tracker-app-contact-service | .NET 8 | Contact management |
| job-tracker-app-journal-service | .NET 8 | Activity journal |
| job-tracker-app-resume-service | .NET 8 | Resume versioning and blob storage |
| job-tracker-app-experience-service | .NET 8 | Work experience document storage and serving |
| job-tracker-app-notification-service | .NET 8 | Async Kafka consumer — processes job create/edit events, sends SMTP email |
| job-tracker-app-ai-service | .NET 8 | AI profile management, server-side prompt assembly, Claude resume generation |
| job-tracker-app-user-service | .NET 8 | User preferences |
| job-tracker-app-web-react | React / TypeScript | Primary frontend (MVP) |
| job-tracker-app-web-angular | Angular / TypeScript | Secondary frontend (Phase 2) |
| job-tracker-app-infrastructure | Docker / Kubernetes / Terraform | Local dev, Docker Compose stack, and Terraform VPS provisioning (Akamai/Linode) |
| job-tracker-app-shared | .NET 8 library | Shared DTOs, events, enums |
| job-tracker-app-docs | Markdown | All documentation |

## Branch strategy

Each repository follows the same branching convention:

- `main` — production-ready code only
- `develop` — integration branch for completed features
- `feature/US-XXX-short-description` — one branch per user story
