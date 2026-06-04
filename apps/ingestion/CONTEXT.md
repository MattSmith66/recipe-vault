# Ingestion

Turns an external Source into one or more standardized Recipes. Owns the messy, source-specific work of extraction and AI-driven standardization, then hands clean Recipes to the Vault.

## Language

**Source**:
An external origin artifact a Recipe is derived from. Subtypes: Cookbook, Website, YouTubeVideo. (Manually-created recipes have no Source and never reach Ingestion — see the Vault.)
_Avoid_: origin, import, provider

**Cookbook**:
A Source subtype — an uploaded PDF that may yield many Recipes (one-to-many).
_Avoid_: book, PDF, collection

**Website**:
A Source subtype — a single web page mapping to a single Recipe (one-to-one).

**YouTubeVideo**:
A Source subtype — a single video whose description or top author comment maps to a single Recipe (one-to-one).

**YouTube Playlist**:
A container Import that enumerates YouTubeVideo Sources — not itself a Source, and carries no cache identity of its own. Each member video is imported and cached independently, so a video imported standalone and the same video inside a playlist share one cache entry.
_Avoid_: playlist source

**Provenance**:
The per-Recipe origin record: a reference to a Source plus the Locus within it and the import event. Retained by Ingestion; not surfaced by the Vault.
_Avoid_: metadata, history

**Locus**:
Where within a Source a Recipe was found — page range (Cookbook), the URL (Website), description or top-comment ID (YouTubeVideo).

**Import**:
One user's submission of one Source for processing. Carries a status and a partial-success result summary ("62 recipes added, 18 had no recipe"); never enters the Vault. Fans out into the standardization of each Potential Recipe the Source yields. Total extraction failure (corrupt PDF, dead URL) fails the Import; some Potential Recipes resolving and others not is success.
_Avoid_: job, upload, scan

**Recipe container**:
An Import that yields multiple Potential Recipes, shown to the user as a parent overview with one row per Potential Recipe. Two shapes that look identical to the user but differ internally: a Cookbook (one Source, many Potential Recipes) and a YouTube Playlist (many YouTubeVideo Sources, one Potential Recipe each).
_Avoid_: bundle, batch

**Recipe standardization**:
Converting messy, source-derived content into the canonical Recipe shape using an AI model.
_Avoid_: parsing, extraction, normalization

**Segment**:
A span of a Cookbook that a deterministic heuristic flags as likely to contain recipe data (keyword cues for ingredients, steps, and nutrition; pages before the first hit are skipped, and a gap page between two hits is folded into the preceding Segment as a continuation). The unit hashed and standardized within a cookbook — one standardizable unit yielding zero or one Standardized Recipe. Heuristic only in V1; no localization.
_Avoid_: section, page, chunk

**Potential Recipe**:
A candidate extracted from a Source that may or may not resolve into a Recipe — not every page, video, or URL contains one. The AI either accepts it (becomes a Standardized Recipe) or rejects it.
_Avoid_: draft, candidate

**Standardization cache**:
A one-to-one mapping keyed by the content hash of a standardizable unit — a one-to-one Source (Website, YouTubeVideo) or an individual Cookbook Segment — to an **optional** StandardizedRecipeId. A null value means the AI found no recipe; there is no separate negative cache, it is the same nullable mapping. A repeat import whose content hashes to an existing key skips the AI call and reuses the Standardized Recipe (or re-confirms its absence). A YouTube Playlist has no entry of its own — it decomposes into member YouTubeVideo Sources, each cached here individually. The StandardizedRecipeId is a logical reference into the Vault, not a database foreign key.
_Avoid_: recipe store, negative cache

**Cookbook cache**:
A separate mapping from a Cookbook's concatenated-content hash to the set of its Segment hashes (one cookbook, many segments). A hit reconstructs the cookbook's recipes by joining those Segment hashes against the Standardization cache and taking the entries whose Standardized Recipe is non-null — skipping both re-segmentation and re-standardization.
_Avoid_: cookbook index
