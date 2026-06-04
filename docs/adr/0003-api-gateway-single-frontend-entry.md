# API gateway as the single frontend entry point

## Status

accepted

## Context

A React SPA must talk to three backend services (Ingestion, Vault, Identity). Exposing each service directly to the browser would replicate auth-token verification, CORS, and rate limiting across all three and leak the internal topology to the client.

## Decision

The SPA talks to one public **API gateway**, which verifies the auth token, applies CORS and rate limiting at the edge, and routes requests to the appropriate service. Backend services are not publicly exposed.

We are *not* adding a backend-for-frontend (BFF). The UI's needs are mostly straightforward per-service calls, and the one aggregation temptation — an Import's status plus the recipes it has produced so far — is handled by the frontend polling until it genuinely warrants one.

## Consequences

- Cross-cutting concerns (auth, CORS, rate limiting) live in one place instead of three services.
- The SPA sees a single origin and is decoupled from internal topology.
- If UI-specific aggregation grows later, a BFF can be introduced behind the same gateway edge without changing the browser contract.
