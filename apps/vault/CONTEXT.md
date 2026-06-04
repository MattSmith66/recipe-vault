# Vault

Stores and serves each user's owned Recipes. The user-facing system of record for browsing, searching, and editing.

## Language

**Recipe**:
The single canonical aggregate — a self-contained dish with title, ingredients, steps, and Nutrition information. User-owned and fully editable. Everything converges to this shape.
_Avoid_: dish, meal, entry

**Standardized Recipe**:
A read-only, ownerless Recipe produced by standardization and kept as the cache artifact. Never browsed or edited directly; copied into a user's vault as a Recipe on every import or cache hit. Stored in a separate table from user Recipes, sharing the Recipe schema.
_Avoid_: template, cached recipe, master recipe

**Manual**:
The origin of a Recipe created directly in the Vault through the editor — no Source, no standardization, no cache. Born user-owned with a null Source label; never passes through Ingestion. Required fields are validated on entry (frontend and Vault backend).
_Avoid_: custom, scratch, authored

**Source label**:
A single nullable string shown alongside a Recipe indicating where it came from — a shareable URL for Website/YouTubeVideo origins, a book title for Cookbook origins, null for Manual. Display-only: the frontend renders a URL as a clickable link and a title as plain text. It is not a grouping or query axis — there is no "browse all recipes from this cookbook." The only Provenance the Vault holds; the full record lives in Ingestion.
_Avoid_: source link, citation, origin

**Nutrition information**:
The intrinsic attributes of a Recipe: serving/yield, calories, and macro distribution. The only recipe attributes in scope.
_Avoid_: metadata, attributes

**Vault**:
The set of Recipes owned by a single user. Each user holds their own copy of a Recipe, editable independently.
_Avoid_: library, collection, cookbook
