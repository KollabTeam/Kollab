# skill commands

Skill commands cover workspace skill discovery, marketplace search, session-scoped install, and space-wide permanent install.

## Querying

```bash
kollab skill list
kollab skill list-official
```

Notes:

- `list` returns the skills visible in the current space
- `list-official` returns the full official recommended skill catalog (not paginated); for tier-3 discovery use `skill search` to keep results focused
- the current CLI automatically uses the authenticated `space_id`, so `--space-id` is handled for you

## Search

```bash
kollab skill search -q "<keywords>"
kollab skill search -q "<keywords>" --page <n> --limit <m>
```

Notes:

- performs a keyword search of the official (system) skill catalog; returns system skills only
- `-q` and `--query` are the same option (case-insensitive substring match on name + description)
- defaults: `--page 1 --limit 20`; max `--limit 50`
- empty `-q` returns the full catalog paginated; pass a specific query to keep results focused
- returns `{items, page, limit, total, has_more}` for result navigation
- use this whenever the goal is to find a specific skill, since it filters the catalog by keyword

## Session-scoped Install (try)

```bash
kollab skill try --name <skill-name>
kollab skill try --id <skill-uuid>
kollab skill try --name <skill-name> --expiration <seconds>
```

Notes:

- hydrates one official skill into the current conversation's workspace via the public backend endpoint `POST /skills/skills/hydrate-from-catalog`
- **session-scoped**: the skill is added only to this conversation's workspace; the space ACL, other conversations, and other users stay as they are
- effect persists across turns within the same conversation via workspace storage sync, and stays local to that conversation
- `try` installs into a Kollab conversation, not this terminal agent. Use it only with an explicit conversation ID; it does not install a local agent skill.
- `--expiration` is optional; omit it for conversation-lifetime scope
- A 409 preloaded-skill response means no remote installation is needed. Continue in that Kollab conversation; do not assume the files exist locally.
- `try` therefore only applies to admin-managed remote (storage-delivered) official skills; `404` means the name/id is not in the official catalog and `400` means the catalog row has no published bundle yet

## Space-wide Permanent Install

```bash
kollab skill install --id <skill-id>
kollab skill install --github-url https://github.com/owner/repo/tree/main/skills/<skill-name>
```

Notes:

- adds the skill to the entire space permanently; affects all members and persists across sessions
- **always ask the user first** via `AskUserQuestion` before running — this mutates the space ACL for everyone
- `--id` installs an official catalog skill by ID
- `--github-url` imports a GitHub skill directory through the managed skill installer, then saves the repository root as source attribution when SKILL.md has not provided richer metadata
- `--id` and `--github-url` are mutually exclusive; they call different server endpoints:
  - `--id` calls `POST /skills/skills/{id}/add-to-space` — installs an official catalog skill by its UUID
  - `--github-url` calls `POST /skills/skills/market/install` — fetches and installs a skill from a GitHub directory URL
- the current CLI covers official install and GitHub import; custom skill management (upload, update, delete, versions, rollback) is covered by the write-side commands below

## Upload (Publish a new version)

```bash
kollab skill upload --file <path/to/skill.zip>
kollab skill upload --file <path/to/skill.zip> --name <override-name>
```

Notes:

- uploads a `.zip` or `.skill` archive and publishes it as the next version for the current space
- the server reads `SKILL.md` inside the archive to extract name, description, and version metadata
- if a skill with the same slug already exists in the space, its version is bumped (one-way ratchet); otherwise a new record is created
- `--name` overrides the filename-derived skill name; if omitted, the filename minus extension becomes the slug

### Identity rule: the filename is the skill (CRITICAL)

The slug comes from the **archive filename** (or `--name`), not from the SKILL.md
frontmatter `name:` — the frontmatter name is only what the skill card displays.
Two archives can display the same name and still be two separate skills.

- to **update** an existing skill, upload under the **same** name/filename it already
  has; the server publishes it as that skill's next version and preserves history
- never upload a `-v2` / `-new` / `-copy` sibling to "keep the old version safe":
  versions are already kept server-side (`skill versions`, `skill rollback`), and a
  different filename creates a SECOND skill the user sees as a duplicate card
- check the current slug with `kollab skill list` before packaging an update; do not
  guess it from a folder name that may have drifted from the frontmatter name
- `kollab skill update` changes metadata only (name / description / enabled). It never
  changes skill content — content updates always mean repackage + `skill upload`
- client-side size guard: 50 MB max; the server accepts whatever storage accepts, so this is the only gate
- calls `POST /api/skills/skills/upload` as `multipart/form-data` with field names `file`, `name`, `space_id`

## Update (metadata / enabled toggle)

```bash
kollab skill update --id <skill-id> --name "New Name"
kollab skill update --id <skill-id> --description "Updated description"
kollab skill update --id <skill-id> --enabled false
kollab skill update --id <skill-id> --enabled true
```

Notes:

- **metadata only** — this command never changes skill content; to ship new content, repackage and `skill upload` under the same name (see the identity rule above)
- partial update: only the fields you supply are changed
- `--space-id <space-id>` overrides `KOLLAB_SPACE_ID` or the current login workspace; enabled changes require a workspace and send body `space_id` for per-space default-installed skill toggles
- `--enabled false` is rejected by the server for system skills (`is_system=true`); they are always enabled and cannot be disabled
- space-private skills can be toggled freely
- at least one of `--name`, `--description`, `--enabled` is required

## Delete

```bash
kollab skill delete --id <skill-id> --confirm
kollab skill delete --id <skill-id> --space-id <space-id> --confirm
```

Notes:

- `--space-id` overrides `KOLLAB_SPACE_ID` or the current login workspace; missing workspace fails locally before any HTTP request
- sends `DELETE /api/skills/skills/{id}?spaceId=<space-id>` for workspace authorization
- permanently removes the skill DB record, all version records, all space-skill config entries, and all storage/local files
- system skills (`is_system=true`) are rejected by the server with 400; retire them globally via platform admins
- requires `--confirm` — this action cannot be undone

## Version History

```bash
kollab skill versions --id <skill-id>
```

Returns `{ skill_id, skill_name, current_version, versions: [{ id, version, s3_location, created_at, change_summary }] }`.
Use the `version` integer from this output as input to `skill rollback`.

## Rollback

```bash
kollab skill rollback --id <skill-id> --version <number> --confirm
```

Notes:

- sets the skill's live version to an earlier published version number
- downloads the target version to the runtime workspace immediately
- requires `--confirm` — this overwrites the current live version for all users in the space
- version number must match a value returned by `skill versions`

## NOT covered via CLI: skill generate / finalize

The AI-generation flow (generate → edit → finalize) is an interactive workflow without a CLI command. Use the Web UI to generate new skills with AI assistance. This limitation is intentional — the CLI is for
managing already-created skills, not for driving the interactive generation workflow.
