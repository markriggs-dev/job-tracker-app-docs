# ADR-007: SignalR transport selection — LongPolling over WebSocket

**Status:** Accepted  
**Date:** 2026-05-09  
**Author:** Mark

## Context

The Job Service Consumer hosts a SignalR hub at `/hubs/jobs`. The React frontend connects to this hub through two reverse proxy layers: Nginx Proxy Manager (NPM) handles TLS termination, and the YARP API Gateway routes the connection to the consumer service. The hub pushes `jobCreated` and `jobUpdated` events to the React client, which invalidates its TanStack Query cache to refresh the UI in real time.

SignalR negotiates a transport in priority order: WebSocket → Server-Sent Events (SSE) → Long Polling. The production deployment stack is:

```
Browser → NPM (nginx, TLS) → YARP API Gateway → Job Service Consumer (Kestrel)
```

## Problem

WebSocket requires each proxy in the chain to correctly forward the HTTP Upgrade handshake and then transparently relay binary frames in both directions. In this deployment:

- NPM successfully upgrades the connection to WebSocket (the browser sees a 101 Switching Protocols)
- YARP receives the upgraded connection and proxies it to Kestrel
- The WebSocket protocol handshake completes at the network level
- However, no data frames flow through the connection — the SignalR handshake message from the client (`{"protocol":"json","version":1}`) never reaches the hub

The result is that the SignalR server-side handshake timeout fires (15 seconds), the server cancels the connection, and the client falls back to SSE, which then also fails. Long Polling is the last fallback, and under the default SignalR hub configuration its poll timeout (90 seconds) exceeded NPM's default `proxy_read_timeout` (60 seconds), causing 504 Gateway Timeout errors and cascading CORS failures on every poll cycle.

Attempts to fix WebSocket:
- Added `app.UseWebSockets()` to the YARP gateway pipeline ✅ (necessary but not sufficient)
- Fixed YARP route from `/hubs/jobs/{**remainder}` to `/hubs/{**remainder}` ✅ (route wasn't matching the hub path without trailing slash)
- Disabled HTTP/2 Support in NPM ✅ (HTTP/2 and WebSocket upgrades are incompatible)
- Added `proxy_read_timeout 300s; proxy_send_timeout 300s;` to NPM custom nginx config — WebSocket frames still did not flow

The double-hop proxy chain (NPM → YARP) is the root cause. Each layer handles the upgrade handshake independently, but frame forwarding through both layers in sequence proved unreliable in this configuration. A single-hop chain (NPM → Kestrel directly) would likely resolve the issue, but that would require removing the API Gateway.

## Decision

Use SignalR's Long Polling transport explicitly, bypassing the WebSocket and SSE fallback attempts. Two fixes were applied to make Long Polling reliable in production:

1. **Hub poll timeout** — `LongPolling.PollTimeout` set to 45 seconds in the hub's `MapHub` options, keeping each poll well under NPM's nginx timeout:

```csharp
app.MapHub<JobsHub>("/hubs/jobs", options =>
{
    options.LongPolling.PollTimeout = TimeSpan.FromSeconds(45);
});
```

2. **NPM proxy timeout** — `proxy_read_timeout 300s` and `proxy_send_timeout 300s` added to the NPM custom nginx configuration, providing headroom for future transport changes and preventing 504s under any load.

3. **Client transport** — forced explicitly in the SignalR HubConnectionBuilder:

```typescript
.withUrl(`${import.meta.env.VITE_GATEWAY_URL}/hubs/jobs`, {
  accessTokenFactory: () => getAccessTokenSilently(...),
  transport: signalR.HttpTransportType.LongPolling
})
```

## Rationale

- Removing the API Gateway to resolve the WebSocket issue would trade a significant architectural talking point for a minor UX improvement. The gateway provides centralized JWT enforcement, a single client-facing origin, and demonstrates API Gateway pattern knowledge.
- Long Polling is a fully supported SignalR transport. It is functionally equivalent to WebSocket for this use case — the only difference is latency between an event occurring and the client receiving it (up to 45 seconds vs. near-instant).
- For a job tracker used by a single authenticated user, a sub-minute refresh delay on job creation and edit events is acceptable. The UI remains consistent and correct; it simply refreshes on the next poll cycle rather than immediately.
- The constraint is documented. Explaining a known infrastructure limitation with a reasoned decision is a stronger portfolio answer than either hiding the issue or over-engineering a fix.

## Alternatives considered

- **Remove the API Gateway:** Eliminates the double-hop and would likely fix WebSocket. Rejected — the gateway is a deliberate architectural choice with portfolio value.
- **Expose the hub on a separate port bypassing YARP:** The browser would connect directly to job-service-consumer through NPM alone (single hop). Viable, but introduces a second NPM proxy host, a second SSL cert path, and splits the React app's API traffic across two origins. Complexity not justified by the benefit.
- **Keep WebSocket with forced LongPolling as fallback:** SignalR's automatic fallback is unreliable here because the WebSocket upgrade completes at the network level before failing silently — SignalR does not immediately detect the failure and fall back. Forcing LongPolling avoids the failed WebSocket attempt entirely and connects cleanly on the first try.

## Consequences

- Real-time updates (job created, job updated) are delivered within the current poll cycle — up to 45 seconds after the event occurs.
- No WebSocket connection is established; browser DevTools will show periodic fetch requests to `/hubs/jobs?id=...` rather than a WebSocket row.
- If the API Gateway is ever removed or replaced with a single-hop proxy, the `transport` override can be removed and SignalR will negotiate WebSocket automatically.
