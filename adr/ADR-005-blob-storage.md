# ADR-005: Blob storage for file management

**Status:** Accepted  
**Date:** 2026-04-11  
**Author:** Mark

## Context

Multiple services need to store binary files uploaded by or generated for users. This includes resume files, work experience documents, and AI-generated resume outputs. A decision is required on how and where these files are stored across services.

## Decision

All binary files are stored in object/blob storage (AWS S3 or Azure Blob Storage in production; MinIO locally). PostgreSQL stores only metadata and a pointer to the storage key. Files are served via each service's API which streams the file directly to the client — the frontend does not use pre-signed URLs that bypass the API.

Three services currently use this pattern:

| Service | Bucket | Content |
|---------|--------|---------|
| Resume Service | `resumes` | User-uploaded PDF/DOCX resume files |
| Experience Service | `experience-profiles` | User-uploaded experience documents (PDF/DOC/DOCX/TXT) |
| AI Service | `generated-resumes` | AI-generated resume Markdown files |

## Rationale

- Binary files should never be stored in a relational database. It creates performance problems, bloats the database, and complicates backups.
- Blob storage is purpose-built for large binary objects and provides durability, redundancy, and scalability at low cost.
- Routing downloads through the service API enforces authentication and authorization checks before any file is served.
- MinIO is an S3-compatible open-source object store that runs in Docker, providing a production-equivalent local development environment.
- A consistent pattern across all three services reduces cognitive overhead and makes the storage layer predictable.

## Alternatives considered

- **Storing files in PostgreSQL as bytea:** Anti-pattern for production systems. Rejected.
- **Pre-signed URLs served directly from storage:** Bypasses the API auth layer. Rejected for this implementation.
- **Local filesystem storage:** Not viable in a containerized environment where pods can be destroyed and recreated.

## Consequences

- Requires an S3-compatible storage bucket (one per service) for production deployment.
- Local development uses MinIO via Docker Compose with separate buckets per service.
- File type validation and virus scanning are noted as future Phase 2 enhancements.
- The Experience Service reads TXT files back as text content for AI prompt assembly; binary formats (PDF/DOCX) require manual attachment by the user in the Claude.AI workflow.
