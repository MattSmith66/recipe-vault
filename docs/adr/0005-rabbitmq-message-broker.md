# RabbitMQ as the message broker

## Status

accepted

## Context

Ingestion distributes standardization work to Celery workers and exchanges messages with the Vault (standardize-and-copy on a cache miss, copy-to-user on a cache hit). The workload is bursty and task-shaped: a cookbook import fans out into hundreds of one-shot jobs, then long-time users import only sporadically. We do not need to retain or replay a message log.

## Decision

Use **RabbitMQ** as the Celery broker and the Ingestion↔Vault message transport. Messages are consume-and-ack: a standardization job is processed once and gone, which matches one-shot tasks. Celery's result backend (for Import status polling) is PostgreSQL (see ADR-0004), not RabbitMQ.

## Considered Options

- **Kafka** — rejected as overkill. It is a retained streaming log built for replay and high-throughput event streams; Celery support is weak, and we have no replay or event-sourcing need. Its strengths are cost here, not value.

## Consequences

- Well-trodden Celery + RabbitMQ path; bursty task fan-out is handled naturally.
- No built-in message replay — acceptable, because jobs are one-shot and idempotency is keyed on content hash (ADR-0001).
- If the system later needs an event log (e.g. source-update propagation, analytics on import streams), that is a separate decision, not a reason to start on Kafka now.
