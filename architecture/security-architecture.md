# Security Architecture — Job Tracker Application

**Version:** 1.0  
**Date:** 2026-05-12  
**Author:** Mark Riggs  
**Status:** Current

---

## Overview

The Job Tracker Application enforces security at the perimeter via a layered defense model: TLS termination at the edge, JWT validation at the API Gateway, and data access scoped to the authenticated user identity at every service. Internal services trust the validated JWT; no backend service is publicly reachable directly.

---

## Security Principles

1. **Perimeter enforcement** — Authentication and TLS are enforced at the network edge before traffic reaches any service.
2. **Identity-scoped data access** — Every database query is filtered by `userId` extracted from the validated JWT. No query returns data belonging to a different user.
3. **Zero direct service exposure** — No backend service is reachable from the internet; all traffic flows through Nginx Proxy Manager → API Gateway → internal Docker network.
4. **Secrets never in source** — Credentials, API keys, and tokens are stored in `.env` files excluded from version control. `.env.example` documents required keys without values.
5. **Minimal attack surface** — VPS firewall drops all inbound traffic except ports 22 (SSH), 80 (HTTP redirect), and 443 (HTTPS). No other ports are publicly reachable.

---

## Identity & Access Management

### Auth0 / OIDC / OAuth2

| Aspect | Detail |
|---|---|
| Provider | Auth0 (external IdP) — eliminates credential storage and session management |
| Protocol | OIDC / OAuth2; Authorization Code flow with PKCE for SPA |
| Token type | JWT (RS256 signed) |
| Token validation | Gateway validates signature against Auth0 JWKS endpoint on every request |
| Token lifetime | Short-lived access tokens; refresh token rotation enabled |
| Scopes | `openid profile email offline_access` |
| Refresh strategy | 401 interceptor in Axios silently retries with a refreshed token; forces logout if session is unrecoverable |

### JWT Validation Flow

```
Client → Nginx Proxy Manager → API Gateway
                                  ├── Validates JWT signature (Auth0 JWKS)
                                  ├── Validates expiry and audience claims
                                  ├── [401 returned if invalid — request dropped here]
                                  └── Forwards bearer token to downstream service
                                            └── Service extracts userId from sub claim
                                                  └── All DB queries filtered by userId
```

### User Identity Propagation

The gateway forwards the validated bearer token to downstream services. Each service extracts the `sub` claim as `userId` and scopes all data operations to that identity. Services never accept a `userId` from the request body — only from the validated token.

---

## Network Security

### Topology

```
Internet
  └── VPS Firewall (DROP all except 22/80/443)
        └── Nginx Proxy Manager — TLS termination (Let's Encrypt)
              └── API Gateway :8080 — JWT validation, CORS, routing
                    └── Internal Docker bridge network (not publicly reachable)
                          ├── Job Service
                          ├── Job Service Consumer (SignalR hub)
                          ├── Contact Service
                          ├── Journal Service
                          ├── Resume Service
                          ├── Experience Service
                          ├── AI Service
                          ├── Notification Service
                          ├── PostgreSQL 16 (no host port mapping in production)
                          ├── Apache Kafka (no host port mapping in production)
                          └── MinIO (no host port mapping in production)
```

### TLS

| Aspect | Detail |
|---|---|
| Certificate authority | Let's Encrypt (auto-renewed via Nginx Proxy Manager) |
| Protocols | TLS 1.2 / 1.3 |
| HTTP redirect | Port 80 redirects to 443; no plaintext traffic accepted in production |
| WebSocket | WSS enforced for SignalR connections |

### Firewall Rules (Akamai VPS — Terraform managed)

| Port | Protocol | Rule |
|---|---|---|
| 22 | TCP | ACCEPT (SSH) |
| 80 | TCP | ACCEPT (HTTP → HTTPS redirect) |
| 443 | TCP | ACCEPT (HTTPS / WSS) |
| All others | — | DROP |

### Internal Network Isolation

All backend services communicate on a private Docker bridge network (`proxy` network). PostgreSQL, Kafka, and MinIO have no host port mappings in production. No internal service is reachable from outside the Docker network.

---

## API Security

| Control | Implementation |
|---|---|
| Authentication | JWT required on all routes; validated at gateway before routing |
| CORS | Allowed origins configured via `Cors:AllowedOrigins` environment variable; no wildcard in production |
| Token forwarding | Bearer token forwarded from gateway to downstream services for service-to-service calls (AI Service → Job Service, AI Service → Experience Service) |
| Input validation | Model validation via ASP.NET Core data annotations; 400 returned on invalid input |
| SQL injection | Mitigated by EF Core parameterized queries; no raw SQL in any service |
| XSS | Rich text input sanitized via DOMPurify before storage; Content-Security-Policy headers planned for Phase 2 |
| File upload | MIME type validation and file extension checks on resume and experience document uploads |

---

## Data Security

### Data at Rest

| Store | Encryption | Notes |
|---|---|---|
| PostgreSQL 16 | Host OS volume encryption (Akamai VPS) | Application-level encryption not implemented; planned for Phase 3 |
| MinIO | Host OS volume encryption (Akamai VPS) | Server-side encryption (SSE-S3) planned for Phase 3 |

### Data in Transit

All client-to-server traffic is TLS-encrypted. Internal Docker network communication (service-to-service, service-to-database) operates on an isolated private network not exposed to the internet.

### Data Classification

| Classification | Examples | Controls |
|---|---|---|
| User content | Job applications, contacts, journal entries, resumes, experience documents | Auth-scoped; `userId`-filtered on every query |
| Generated output | AI-generated resume files (MinIO) | Auth-scoped; `userId`-filtered; download via signed storage key |
| Credentials | Auth0 secrets, SMTP passwords, API keys, DB passwords | `.env` only; never in source control; `.env.example` documents shape |
| PII | Email address (from Auth0 JWT claim) | Not persisted in application database; sourced from JWT claim at runtime only |

---

## Secrets Management

### Current State

| Secret | Storage | Access Pattern |
|---|---|---|
| Auth0 Client Secret | `.env` (gitignored) | Read at startup via `IConfiguration` |
| Auth0 Domain / Audience | `.env` (gitignored) | Read at startup via `IConfiguration` |
| Anthropic API Key | `.env` (gitignored) | Injected into `AnthropicClient` at startup |
| SendGrid SMTP credentials | `.env` (gitignored) | Read at startup by Notification Service |
| PostgreSQL password | `.env` (gitignored) | EF Core connection string |
| MinIO access/secret key | `.env` (gitignored) | MinIO SDK client |
| DuckDNS token | `.env` (gitignored) | DuckDNS updater container |

`.env.example` is committed to every repository and documents all required keys without values.

### Phase 3 Target

HashiCorp Vault or Azure Key Vault for dynamic secret rotation and removal of `.env` files from the deployment model. This eliminates the risk of secret exposure via misconfigured file permissions or accidental inclusion in container images.

---

## Threat Model

| Threat | Vector | Mitigation | Status |
|---|---|---|---|
| Unauthenticated API access | Direct HTTP call bypassing auth | JWT validation at gateway; 401 returned before routing | Mitigated |
| Cross-user data access | Authenticated user querying another user's data | `userId` claim scoped on every DB query; never accepted from request body | Mitigated |
| Credential exposure via source control | Secrets committed to git | `.env` gitignored; `.env.example` documents keys without values | Mitigated |
| Man-in-the-middle | HTTP interception | TLS 1.2/1.3 enforced; HTTP → HTTPS redirect; WSS for WebSocket | Mitigated |
| Direct service bypass | Request to internal service bypassing gateway | Services on isolated Docker bridge network; no host port mappings | Mitigated |
| Brute force / credential stuffing | Repeated auth attempts | Auth0 anomaly detection and rate limiting; application never handles credentials | Delegated to IdP |
| Token replay after expiry | Reuse of expired JWT | Short-lived tokens; expiry validated on every request at gateway | Mitigated |
| XSS via rich text input | Malicious script in journal/job description fields | DOMPurify sanitization on all rich text input before storage | Mitigated |
| SQL injection | Malicious input in query parameters or body | EF Core parameterized queries; no raw SQL | Mitigated |
| Malicious file upload | Executable or oversized file via resume/experience upload | MIME type and extension validation; size limits enforced | Mitigated |
| Container escape | Compromised container accessing host | Docker default isolation; non-root container users where possible | Partial |
| Secrets in Kubernetes (Phase 3) | `.env` files in K8s pod specs | HashiCorp Vault or Azure Key Vault planned | Planned (Phase 3) |
| Encryption at rest | Disk-level access to PostgreSQL/MinIO data | Application-level encryption not yet implemented; host-level encryption present | Accepted / Phase 3 |

---

## Compliance Considerations

This is a portfolio application. The following compliance postures are noted for architectural context.

| Regulation | Relevance | Current Posture |
|---|---|---|
| GDPR | Email address from Auth0 JWT; user-generated content | PII not persisted in app DB; user data deletable on request | Partial |
| SOC 2 | Logging, access controls, change management | Access controls in place; logging planned (Phase 2); CI enforces change gates | Partial |
| OWASP Top 10 | Web application vulnerability coverage | Auth, injection, XSS controls in place; CSP headers and server-side encryption pending | Partial |

---

## Related Documents

- [Reference Architecture](reference-architecture.md)
- [Engineering Standards & Governance](engineering-standards-governance.md)
- [Technology Roadmap](technology-roadmap.md)
- [ADR-003 — Auth0 for Federated Identity](../adr/ADR-003-auth0-identity.md)
- [ADR-009 — Terraform for VPS Provisioning](../adr/ADR-009-terraform-vps-provisioning.md)
