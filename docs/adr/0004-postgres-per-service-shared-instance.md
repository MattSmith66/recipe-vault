# PostgreSQL per service on a shared MVP instance

## Status

accepted

## Context

Each service (Ingestion, Vault, Identity) needs persistence with separate datastores so the context boundary holds (ADR-0001 relies on cross-boundary pointers being logical references, not enforced foreign keys). We also want to keep MVP infrastructure cheap, and the Vault needs to search recipes by title and ingredient.

## Decision

- **PostgreSQL** is the datastore for every service. No workload here needs anything more specialised.
- For the MVP, the services share **one PostgreSQL instance** as **separately-credentialed databases** (distinct databases and roles), not three instances. The logical boundary is enforced by allowing **no cross-database queries or foreign keys** — so the deployment can later split into separate instances with no code change.
- **Vault search uses PostgreSQL full-text search** (`tsvector` / GIN) for V1. Search is per-user over that user's own vault — a small corpus that Postgres handles comfortably.
- Celery's **result backend** (needed for Import status polling) is PostgreSQL for the MVP, to avoid running a separate Redis.

## Considered Options

- **ElasticSearch for Vault search** — rejected for V1: per-user corpora are small, and ES adds a cluster, a sync pipeline, and Postgres↔ES eventual consistency. Recorded as a future option if search outgrows Postgres FTS.
- **A separate Postgres instance per service from day one** — deferred: correct end state, but unnecessary cost for an MVP given the boundary is already enforced logically.

## Consequences

- Cheap MVP footprint; clean path to per-service instances later.
- The "no cross-database FK/query" rule must be respected, or the shared instance silently couples the services.
- Using Postgres as the Celery result backend keeps the service count down at some cost to task-state read performance — revisit if polling load grows.
