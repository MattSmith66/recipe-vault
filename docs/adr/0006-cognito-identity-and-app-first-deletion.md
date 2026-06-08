# Cognito user pool as the identity directory; app-first account deletion

## Status

accepted

## Context

ADR-0003 names the API gateway as the component that verifies the auth token, but the auth mechanism itself was deferred (`apps/identity/CONTEXT.md`). We are hosting on AWS, so an Amazon Cognito **user pool** is the concrete identity provider: it authenticates users and issues JWTs whose `sub` claim is a stable per-user UUID.

This forces two linked decisions. Should we continue using an identity service or simply go with Cognito? Second account deletion must cascade to every context that owns user-scoped data, but **Cognito provides no "user deleted" Lambda trigger.** The only native deletion signal requires a CloudTrail log to be forwarded to EventBridge on a best-effort, unordered basis, two AWS services we are not currently using.

## Decision

- **Cognito user pool is the identity directory.** There is no standalone Identity service or Identity database. The Identity context survives only as *language*: it defines `User` and `UserId`. **`UserId` is the Cognito `sub`.** Each service stores that `UserId` on the rows it owns; the gateway extracts `sub` from the verified access token and forwards it downstream (see ADR-0003).

- **Account deletion is app-first.** The only deletion path is a "delete my account" command through the gateway. The handler (a) emits an `AccountDeletionRequested` domain event, then (b) calls Cognito `AdminDeleteUser`. **Cognito self-service deletion (hosted-UI) is disabled** so Cognito can never be the originator. Deletion is therefore *our* transaction, not a best-effort event we must catch.

- **Propagation and cleanup reuse existing infrastructure.** `AccountDeletionRequested` is published over RabbitMQ (ADR-0005) to a durable queue. Each context that owns user-scoped data consumes it and **soft-deletes** that user's data, opening a **30-day undo window**. A scheduled sweep (CRON / DB procedure) **hard-deletes** records whose window has elapsed. Cross-context cleanup is event-driven by necessity: there are no cross-service foreign keys to cascade through (ADR-0004).

## Considered Options

- **Identity-as-local-mirror** (a thin Identity table keyed by `sub` for vendor decoupling) — rejected. With no groups, sharing, or app-specific profile data in scope, a mirror earns nothing today.

- **Cognito-first / event-driven deletion** (Cognito deletion -> CloudTrail -> EventBridge -> Lambda -> RabbitMQ) — rejected as the primary path. CloudTrail to EventBridge delivery is best-effort and unordered, so a dropped event orphans a vault permanently; making it safe would require a periodic reconciliation sweep (diff Cognito users against stored `UserId`s) purely as a backstop. App-first removes the dependency entirely. Also unnecessarily complicated when not using these services currently.

## Consequences

- The Identity service disappears as a deployable unit; "who is this user" is answered by Cognito + the gateway's `sub` extraction. Anyone expecting an Identity database will not find one — that absence is deliberate.
- We are coupled to Cognito as the auth provider. Because the domain only ever references `UserId`, a future migration changes the token issuer and the `sub` source, not the contexts.
- Deletion correctness depends on disabling Cognito self-service delete. If a user is ever removed out-of-band (admin console, IaC), no `AccountDeletionRequested` is emitted and that user's data is orphaned — the EventBridge listener + reconciliation sweep from the rejected option becomes the fallback to add **if** out-of-band deletion ever becomes possible.
- Each user-scoped context must implement the soft-delete + window + sweep; the 30-day undo is a product-visible guarantee that lives in every owning service, not one place. Since app scope is simple, this should only impact the vault service.
