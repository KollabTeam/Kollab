# artifact commands

Artifact commands cover conversation-, project-, and space-level lookup plus common artifact actions.

## Querying

```bash
# Conversation / project default to including workspace-sync artifacts (matching task UI / Project Library).
kollab artifact list --conversation-id <conversation-id>
kollab artifact list --project-id <project-id>

# Opt out of workspace-sync inclusion for conversation / project:
kollab artifact list --conversation-id <conversation-id> --no-include-workspace-sync

# Space listing excludes workspace-sync by default (matching Space Library). Opt in:
kollab artifact list --space-id <space-id> --include-workspace-sync --page 1 --page-size 20

kollab artifact url --id <artifact-id>
```

Notes:

- exactly one of `--conversation-id`, `--project-id`, or `--space-id` must be provided
- space-level listing supports pagination
- conversation and project scopes include workspace-sync artifacts by default; use `--no-include-workspace-sync` to exclude
- space scope excludes workspace-sync by default; use `--include-workspace-sync` to include

## Common Actions

```bash
kollab artifact save-to-space --id <artifact-id> --parent-block-id <block-id>
kollab artifact install-skill --id <artifact-id>

# update-content: replace full UTF-8 content only when the read version is still current.
# Use --content for short inline strings.
# Use --file for project notes or any content that would exceed shell-arg length limits.
# Both paths enforce a 2MB client-side guard that matches the server cap.
kollab artifact update-content --id <artifact-id> --content "new content" --expected-version <content-version>
kollab artifact update-content --id <artifact-id> --file /path/to/note.md --expected-version <content-version>

# get-content: fetch and print raw artifact content to stdout (capped at 2MB).
# Resolves the download URL via GET /artifacts/{id}/url then fetches it.
# Output is raw bytes, not JSON — pipe-friendly for agents reading project notes.
# Errors are reported in the standard JSON envelope on stderr.
kollab artifact get-content --id <artifact-id>

# Conflict-safe read-modify-write: --json returns data.content, data.content_version, and data.truncated.
# Never update when data.truncated=true because the complete document was not loaded.
# Otherwise use that exact version. HTTP 409 means another writer saved first; re-read and reconcile.
kollab artifact get-content --id <artifact-id> --json

kollab artifact delete --id <artifact-id>
```

## Share Management

A 32-char hex share token grants anonymous read access at `<api-host>/artifact/<shareToken>`
(production: `https://kollab.im/artifact/<shareToken>`). Requires manage access — owner, or space
admin/owner.

```bash
# Read current share state (returns shareToken and the full public shareUrl)
kollab artifact share get --id <artifact-id>
# { "shareToken": "abc123..." | null, "shareUrl": "https://kollab.im/artifact/abc123..." | null }

# Enable sharing idempotently (reads state first, toggles only if disabled)
kollab artifact share enable --id <artifact-id>

# Disable sharing idempotently (reads state first, toggles only if enabled)
kollab artifact share disable --id <artifact-id>

# Raw toggle (always toggles, matching the server PUT)
kollab artifact share toggle --id <artifact-id>
```

Notes:

- A fresh share token is generated on each re-enable (previous tokens expire immediately).
- `enable` / `disable` are idempotent and safe to retry.
- The public URL is `<api-host>/artifact/<shareToken>` (e.g. `https://kollab.im/artifact/<token>`).

## Article Theme (Markdown Artifacts)

Cover image, background, font family, and font size for a Markdown artifact. Changes persist
server-side immediately and are reflected on the share page.

```bash
# Read current theme ({ articleTheme } — null when unset)
kollab artifact theme get --id <artifact-id>

# Set theme fields (partial update — only specified fields change)
kollab artifact theme set --id <artifact-id> \
  [--cover <url|search:query>] [--cover-none] [--cover-preset <group>/<0-5>] [--cover-search-index <n>] \
  [--background <preset-id>] \
  [--background-image <url|search:query>] [--background-image-none] [--background-search-index <n>] \
  [--font <family>] [--size <size>]

# Search Unsplash images for use as cover or background
kollab artifact theme search-image --query "mountain landscape" [--page 1] [--limit 24]
# { "results": [{ "id", "thumbUrl", "fullUrl", "authorName", "downloadLocation", ... }], "total", "totalPages" }

# List all built-in backgrounds, fonts, sizes, and cover preset groups (no network call)
kollab artifact theme list-presets
```

Valid values:

- `--background`: `none | paper-light | paper-dark | canvas-blue | linen-warm | gradient-dusk`
- `--font`: `sans | serif | mono`
- `--size`: `sm | base | lg`
- `--cover-preset` groups: `illustration | doodle | landscape | solid-color | oil-painting | abstract` (index 0-5)

Notes:

- `search:<query>` (in `--cover` / `--background-image`) runs an Unsplash search, picks the
  result at `--cover-search-index` / `--background-search-index` (default 0), sets the URL +
  photographer attribution, and fires `POST /api/artifacts/unsplash/track-download`
  (Unsplash TOS §3.b). It is fire-and-forget: it never blocks or fails the theme update.
- `--cover-preset` resolves a built-in image offline and also fires `track-download`.
- These commands work identically under `kollab workspace artifact share/theme ...`.

## Agent workflow example

Style an article, publish it, and confirm what was stored:

```bash
# 1. Style the article
kollab artifact theme set --id $ARTIFACT_ID \
  --cover "search:technology abstract blue" --background paper-light --font serif --size base
# 2. Enable sharing and get the public URL
kollab artifact share enable --id $ARTIFACT_ID   # prints shareUrl
# 3. Verify
kollab artifact theme get --id $ARTIFACT_ID
```
