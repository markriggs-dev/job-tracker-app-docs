# ADR-005: Blob storage for resume file management

**Status:** Accepted  
**Date:** 2026-04-11  
**Author:** Mark

## Context

The Resume Service needs to store resume files uploaded by users. Files can be PDFs or Word documents. A decision is required on how and where resume files are stored.

## Decision

Resume files will be stored in object/blob storage (AWS S3 or Azure Blob Storage in production; MinIO locally). PostgreSQL stores only the metadata and a pointer to the file location. Files are served via the Resume Service API which streams the file directly to the client — the frontend does not use pre-signed URLs that bypass the API.

## Rationale

- Binary files should never be stored in a relational database. It creates performance problems, bloats the database, and complicates backups.
- Blob storage is purpose-built for large binary objects and provides durability, redundancy, and scalability at low cost.
- Routing downloads through the Resume Service API enforces authentication and authorization checks before any file is served.
- MinIO is an S3-compatible open-source object store that runs in Docker, providing a production-equivalent local development environment.

## Alternatives considered

- **Storing files in PostgreSQL as bytea:** Anti-pattern for production systems. Rejected.
- **Pre-signed URLs served directly from storage:** Bypasses the API auth layer. Rejected for this implementation.
- **Local filesystem storage:** Not viable in a containerized environment where pods can be destroyed and recreated.

## Consequences

- Requires an S3-compatible storage bucket for production deployment.
- Local development uses MinIO via Docker Compose.
- File type validation and virus scanning are noted as future Phase 2 enhancements.
