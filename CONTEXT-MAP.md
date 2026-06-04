# Context Map

## Contexts

- [Ingestion](./apps/ingestion/CONTEXT.md) — turns external Sources into standardized Recipes
- [Vault](./apps/vault/CONTEXT.md) — stores and serves each user's owned Recipes for browsing, search, and editing
- [Identity](./apps/identity/CONTEXT.md) — owns who a User is; referenced elsewhere only by `UserId`

## Relationships

- **Ingestion → Vault**: Ingestion standardizes a Potential Recipe into a Standardized Recipe (stored in the Vault) and asks the Vault to copy it into the importing user's vault as a user-owned Recipe. On a repeat import of an unchanged Source, Ingestion's Standardization cache hits and only the copy happens — no AI call.
- **Identity → Vault / Ingestion**: Both reference a User by `UserId` only. Vaults are strictly per-individual; no sharing. See [ADR-0002](./docs/adr/0002-per-individual-vaults-no-sharing.md).
- **Ownership split**: The Vault owns all recipe *content* — both Standardized Recipes and user Recipes. Ingestion owns the Standardization cache (a nullable `content hash → StandardizedRecipeId` mapping) and the Cookbook cache (`cookbook hash → set of Segment hashes`), plus Source/Locus/Provenance. Cross-boundary pointers are logical references (id values), not database foreign keys. See [ADR-0001](./docs/adr/0001-copy-on-import-from-standardized-recipe-cache.md).
