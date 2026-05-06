# ADR-006: React as the initial frontend framework, with Angular as a planned second implementation

**Status:** Accepted  
**Date:** 2026-04-11  
**Author:** Mark

## Context

The application requires a dynamic single-page application with complex UI state — multi-step forms, status boards, activity journals, and file upload flows. A decision is required on the frontend framework. The architecture should also support a second frontend implementation to demonstrate framework-agnostic API design.

## Decision

React will be used as the initial frontend framework. An Angular implementation will follow as a second frontend, consuming the same API gateway. Both frontends will be built as SPAs served as static builds, with all API communication handled via HTTPS calls to the API gateway. The API layer will remain completely frontend-agnostic.

## Rationale

- React is the most widely used frontend framework in enterprise environments, making it the most recognizable starting point for a portfolio application.
- Building a second Angular frontend against the same API directly demonstrates that the backend was designed correctly — frontend-agnostic, contract-driven, and decoupled from any presentation layer.
- The dual-frontend approach is a strong portfolio differentiator. It shows breadth across both major enterprise frameworks and reinforces the architectural principle of separation of concerns.
- Auth0 provides well-supported SDKs for both React and Angular, enabling consistent authentication implementation across both frontends.
- React and Angular are the two most commonly required frameworks in enterprise engineering job descriptions, making both implementations directly relevant to the job search.
- Starting with React allows faster initial progress while the Angular implementation follows as a Phase 2 effort.

## Alternatives considered

- **Vue.js:** Lightweight and approachable but less prevalent in enterprise environments. Not selected as it would not add the same portfolio value as Angular.
- **Server-side rendering (Next.js):** Adds complexity not justified for an authenticated single-user application.
- **Angular only:** Would miss the opportunity to demonstrate React experience, which has broader market coverage.

## Consequences

- The API gateway and all microservices must expose clean REST contracts with no assumptions about the consuming frontend.
- Both frontends will need separate build pipelines configured in GitHub Actions.
- Environment-specific configuration (API URLs, Auth0 tenant) is managed via environment variables at build time for both implementations.
- The Angular implementation will be tracked as a Phase 2 deliverable once the React frontend reaches feature parity.
- Maintaining two frontends increases long-term maintenance surface but the portfolio and learning value justifies this for the current use case.
