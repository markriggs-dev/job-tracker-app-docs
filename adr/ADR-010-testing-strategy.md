# ADR-010 — Testing Strategy

**Date:** 2026-05-11
**Status:** Accepted

## Context

The project targets senior EM / Solution Architect roles. Job descriptions in that space consistently call out end-to-end ownership that includes unit tests alongside API development and UI delivery. A codebase without any test coverage is a portfolio gap, even when the functional product works correctly.

The key constraints are:
- Multiple .NET microservices with service-layer business logic worth testing
- React/TypeScript frontend built on Vite
- No CI test gate existed — the deploy pipeline ran even if code was broken
- Integration tests require a live database and Kafka; those are out of scope for automated CI at this stage

## Decision

### .NET services — xUnit unit tests

Use **xUnit** with **Moq** (mocking) and **FluentAssertions** (readable assertions) for all service-layer unit tests. These were already referenced in the `JobService.UnitTests.csproj` scaffold.

The service layer (e.g. `JobRequisitionService`) is the right test target:
- It contains business logic that can fail silently (e.g. the `DateSubmitted` auto-fill rule)
- Its dependencies (`IJobRequisitionRepository`, `IJobEventPublisher`) are interface-backed and trivially mockable
- Repository and controller layers are thin wrappers — covered implicitly

The existing CI workflow (`ci.yml`) in each service repo already runs `dotnet test`. Tests now run automatically on every push to `develop` and `main`, and block the pipeline on failure.

### React frontend — Vitest

Use **Vitest** (Vite-native, zero config) with **@testing-library/jest-dom** for DOM assertions. Pure utility functions are the primary test target — they require no component mounting, Auth0 mocking, or HTTP interception.

Status-classification logic (which statuses imply a submission, date formatting) was extracted into `src/utils/jobUtils.ts` as a natural test surface. This also removed a duplication between `DashboardPage` and future callers.

The deploy workflow (`deploy.yml`) now runs `npm run test:run` before the build step, so a failing test blocks the GitHub Pages deployment.

### What is not tested (and why)

| Layer | Reason skipped |
|---|---|
| Repository / EF Core | Requires a real PostgreSQL connection — integration test scope |
| API controllers | Thin; no logic beyond routing to the service layer |
| Kafka consumers | Require a broker — integration test scope |
| React components with Auth0 / React Query | Heavy provider mocking for low signal; UI correctness verified manually |

## Alternatives considered

**No tests** — Rejected. Even a small test suite demonstrates the discipline and catches regressions on the logic that matters most.

**Integration tests in CI** — Deferred. Would require Docker-in-Docker (PostgreSQL + Kafka) in the GitHub Actions runner. Viable future addition with `services:` containers in the workflow YAML.

**100% coverage target** — Rejected. Coverage as a target optimises for the metric, not the outcome. Tests focus on business logic paths that have a meaningful failure mode.

## Consequences

- The `DateSubmitted` auto-fill logic and status-classification rules are now regression-protected
- Any future change to `JobRequisitionService` that breaks existing behaviour will fail CI before it reaches `main`
- The pattern is established — adding tests to other services (Contact, Journal, Resume, AI) follows the same structure
