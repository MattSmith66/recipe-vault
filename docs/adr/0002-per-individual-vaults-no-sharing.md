# Per-individual vaults; no recipe sharing

## Status

accepted

## Context

The vault imports recipes from purchased cookbooks as well as public websites and YouTube. Sharing recipe *content* between users would republish copyrighted material — most acutely, recipes from purchased cookbooks — to people who never bought it.

## Decision

Vaults are strictly per-individual User. There are no shared or household vaults, no cross-user recipe sharing, and no social or discovery surface in V1. Each user imports and owns a private copy of every recipe (copy-on-import).

The only outward sharing is a nullable **Source link** — the originating web or YouTube URL — displayed alongside a Recipe, which a user may share so others can import the recipe themselves. Cookbook and Manual recipes have no Source link.

## Consequences

- Copyright exposure stays minimal: purchased cookbook content never leaves the importer's private vault and has no shareable link. "No link" is the firewall expressed in the data model.
- No concurrent-editor or group-ownership problem — "owner = one User, fully editable" holds.
- Even sharing a user's *own* recipes stays off: editing an imported recipe yields a user-owned copy indistinguishable from a hand-authored manual one, so the system cannot tell an original recipe from a lightly-edited copyrighted one. Offering "share your own recipes" would require a provenance distinction the Vault deliberately omits.
- Shared/household vaults and social/discovery are deferred and would reopen the ownership model.
