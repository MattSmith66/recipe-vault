# Copy-on-import from a Standardized Recipe cache

## Status

accepted

## Context

Recipes enter from messy, source-specific inputs (PDF cookbooks, websites, YouTube descriptions/comments) that an AI model standardizes into the canonical Recipe shape. AI standardization is the expensive step, so we avoid re-running it on a Potential Recipe we have already standardized. We also want each user to fully own and edit their recipes, and the Recipe schema to live in exactly one place.

## Decision

Standardization produces a **Standardized Recipe**: a read-only, ownerless row stored in the **Vault** (in a table separate from user-facing recipes, sharing the Recipe schema). On every import — whether a fresh standardization or a cache hit — the Vault makes a **user-owned, editable copy** of each Standardized Recipe and places it in the importing user's vault.

The **Vault** owns recipe content; **Ingestion** owns the caches:

- **Standardization cache** — a one-to-one mapping from the content hash of a Potential Recipe (a one-to-one Source, or a Cookbook Segment) to an *optional* `StandardizedRecipeId`. A null value records "no recipe found"; there is no separate negative cache.
- **Cookbook cache** — a mapping from a Cookbook's concatenated-content hash to the set of its Segment hashes. A hit reconstructs the book's recipes by joining those Segments against the Standardization cache and taking the non-null ones, skipping both re-segmentation and re-standardization.

`StandardizedRecipeId` is a logical reference into the Vault, not a database foreign key (the services have separate datastores). Standardized Recipes are never purged, so the reference is stable.

## Messaging (Ingestion ↔ Vault)

- **Cache miss (new Standardized Recipe):** the Ingestion worker standardizes the Potential Recipe, then sends the standardized content plus the importer's `UserId` to the Vault. The Vault creates the read-only Standardized Recipe *and* the user-owned copy, then replies with the new `StandardizedRecipeId` so Ingestion can seed its Standardization cache. An AI rejection writes `hash → null` in Ingestion only — no Vault message.
- **Cache hit:** Ingestion sends the importer's `UserId` and an array of `StandardizedRecipeId`s (several for a cookbook, a single-element array otherwise). The Vault copies each into the user's vault. No reply is needed — the cache is already populated.

## Considered Options

- **Per-user copies with a separate standardized-blob cache** — the cache stores recipe-shaped content outside the Vault. Rejected: duplicates the Recipe schema across the Ingestion/Vault boundary.
- **Shared read-only recipes referenced by users, forked copy-on-write on edit** — no duplicate rows. Rejected: turns the Vault's ownership into a reference/membership relation with two recipe modes that every feature (browse, search, edit, delete) must handle.

## Consequences

- Each unique Potential Recipe is standardized by the AI at most once; further imports are cache hits that only copy.
- User-facing recipes stay simple: one owner, fully editable, no mode checks — the read-only/owned split is a clean cache-vs-user-facing table partition.
- Storage holds duplicate copies of popular recipes. Accepted: recipe payloads are tiny text; the duplication cost is negligible against the AI cost saved.
- The content hash must be treated idempotently so two concurrent first-imports of the same Source don't create duplicate Standardized Recipes.
- Pushing later Source updates to existing user copies is harder than a reference model would make it (every derived copy must be found and offered a re-copy). Deferred — this feature may not ship.
