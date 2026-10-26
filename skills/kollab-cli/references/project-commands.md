# project commands

Project commands cover project CRUD, document search and reading, block operations, and project collaboration permissions.

## Basic CRUD

```bash
kollab project list
kollab project items --id <project-id>
kollab project create --title "Docs"
kollab project update --id <project-id> --title "New Name"
kollab project update --id <project-id> --description "Everything about onboarding"
kollab project delete --id <project-id> --confirm
```

## Documents and Search

```bash
kollab project documents --project-id <project-id> --page 1 --page-size 50
kollab project read --file-id <file-id> --page 1 --page-size 5000
kollab project search --query "session storage" --project-id <project-id> --expand-nodes 1 --score-threshold 0.45
kollab project search --query "session storage" --project-id <project-id> --top-k 20
```

Notes:

- `--project-ids` and `--file-ids` accept comma-separated values or JSON array strings
- `documents` and `search` also accept singular `--project-id <uuid>` (merged into `--project-ids`), matching the option name used by every other project subcommand
- `search` supports `--top-k` (default 10), `--expand-nodes` (default 1), and `--score-threshold` (0–1, default 0.3)
- for project details, use `items`, `documents`, `read`, or the access endpoints

## File Upload

```bash
kollab project upload \
  --project-id <project-id> \
  --file /path/to/document.pdf

kollab project upload \
  --project-id <project-id> \
  --file /path/to/report.docx \
  --title "Q1 Report" \
  --parent-id <folder-block-id>
```

Notes:

- Uploads a local file into the knowledge base using a 3-step sequence: request a presigned storage URL, PUT the file bytes directly to storage, then create the knowledge-base block entry.
- `--project-id` is required; `--parent-id` defaults to the project root if omitted; `--title` defaults to the local filename.
- Maximum file size: **500 MB** (aligned with the platform limit). Larger files require splitting first.
- The response includes `blockId`, `title`, `resourceId`, `existed` (true when the hash-based dedup skipped the storage upload), and `processStatus`.
- **Vectorization is asynchronous**: after `upload` returns, the server parses and embeds the file in the background. Use `kollab project documents` or wait a few seconds before running knowledge queries against the newly uploaded file.
- Supported types include PDF, DOCX, TXT, MD, CSV, JSON, HTML, PNG, JPG, MP3, MP4, ZIP, and more; unrecognised extensions are stored as `application/octet-stream`.

## Block Operations

```bash
kollab project create-link --project-id <project-id> --title "Spec" --url https://example.com
kollab project create-folder --project-id <project-id> --title "Notes"
kollab project update-block --block-id <block-id> --title "Renamed"
kollab project delete-file --block-id <block-id>
kollab project file-url --file-id <file-id>
```

### Moving Blocks

`update-block --parent-id` is the move operation. There is no separate `move-block` command
because the server's `PATCH /knowledge-bases/blocks/:uuid` endpoint (`UpdateBlockDto.parentId`)
handles cross-project and cross-folder reparenting.

```bash
# Move block to a new parent folder or project root
kollab project update-block --block-id <block-id> --parent-id <target-folder-id>
```

Notes:

- `create-link` is the **Add-Resource** entry point: it creates a LINK block and auto-submits the URL for processing. The backend fetches the content (social/media transcript via the bridge — YouTube / Bilibili / TikTok / Douyin / Instagram / X / Facebook / WeChat MP / Zhihu / Weibo / Xiaohongshu / LinkedIn / Threads / Reddit / Lemon8 / Toutiao / podcast / direct media file — or scrapes a generic webpage), converts it to text/Markdown, chunks it, and writes dense + bm42 vectors to the knowledge base. So a created link becomes searchable by `kollab knowledge ask/search/keyword-search` once processing completes.
- `--title` and `--url` are both required by the CLI; pass the URL as the title when you have no better label — after processing, the resource card displays the FETCHED content title, not the raw URL.
- Manual `create-link` runs no agent and therefore spends no model-token credit. A Tuner-backed provider
  ingestion can still debit the Project workspace for its positive upstream provider cost; cache hits add no
  provider debit, and the server's idempotent ledger is the billing source of truth.
- `create-folder` groups resources; `update-block` renames; `delete-file` removes a resource (its vectors leave the active set); `file-url` returns a download/preview URL for a file block.

## Project Manual Artifacts

```bash
kollab project create-manual-artifact --project-id <project-id> --filename plan.md [--content "# Plan" | --file /tmp/seed.md]
```

- Creates a Markdown manual artifact owned by the project (`sourceType=manual`), not a knowledge-base block: it is NOT vectorized and does not appear in `knowledge` search results. It shows in the project artifacts panel and the composer `@` mention menu.
- `--content` / `--file` are optional (empty manual artifact by default) and mutually exclusive; content is capped at 2MB.
- Read a manual artifact with `kollab artifact get-content --id <artifact-id> --json`; stop if `data.truncated=true`, otherwise keep `data.content_version` and conditionally overwrite it with `kollab artifact update-content --id <artifact-id> --file /tmp/updated.md --expected-version <content-version>`. HTTP 409 means another collaborator saved first: re-read and reconcile instead of retrying stale full content. Stage update content under `/tmp`, never inside the task workspace (a workspace copy would sync back as a duplicate task artifact).
- List a project's manual artifacts (with their artifact ids) via `kollab artifact list --project-id <project-id>`.

## Collaboration Permissions

```bash
kollab project access --id <project-id>
kollab project invite --id <project-id> --email user@example.com
kollab project invite-link --id <project-id>
kollab project remove-member --id <project-id> --user-id <user-id>
```

## Block Permissions

These commands manage fine-grained read/write/manage access on any block (project root, folder, or file).

```bash
# Inspect permissions on a block
kollab project block-permission-list --block-id <uuid>

# Add a permission item (requires manage access on the block)
kollab project block-permission-add --block-id <uuid> --type user --user-id <uid> --role writer
kollab project block-permission-add --block-id <uuid> --type space --role reader
kollab project block-permission-add --block-id <uuid> --type group --group-id <gid> --role reader

# Remove a permission item (body-based — confirmed from server DTO)
kollab project block-permission-remove --block-id <uuid> --type user --user-id <uid>
kollab project block-permission-remove --block-id <uuid> --type group --group-id <gid>
kollab project block-permission-remove --block-id <uuid> --type space

# Check the caller's own resolved role on a block
kollab project block-permission-role --block-id <uuid>
```

Notes:

- `--type` values: `user` | `group` | `space` | `public` | `restricted` | `illegal`
- `--role` values: `none` | `reader` | `commenter` | `writer` | `editor`
- `--user-id` is required with `--type user`; `--group-id` is required with `--type group` (both on add and remove)
- `block-permission-add --is-guest` exists on the command but is not supported on project blocks — do not pass it here
- For project-type blocks (knowledge bases), only `type=user` is supported and the server
  accepts `writer` (member) or `editor` (owner). Other type/role combinations return 400.
- `block-permission-remove` sends `userId`/`groupId` in the request **body** (not query params).
- `block-permission-role` returns `{ blockId, role, joinedSpace }` where `joinedSpace` indicates
  whether the caller is a space member; inherited PUBLIC/RESTRICTED rules depend on this.

## File Processing Status

After `project upload` or `project create-link`, files are processed asynchronously. Poll status
until `processStatus` is `completed` before running knowledge search.

```bash
# Single file
kollab project file-status --file-id <block-uuid>

# Batch (up to 100 file UUIDs; accepts CSV or JSON array)
kollab project file-status --file-ids "uuid1,uuid2,uuid3"
kollab project file-status --file-ids '["uuid1","uuid2"]'
```

Common `processStatus` values: `pending` | `uploaded` | `processing` | `completed` | `failed`

Typical poll-after-upload flow:
```bash
# 1. Upload
kollab project upload --project-id <project-id> --file /path/to/doc.pdf
# 2. Poll until completed (or use --file-ids for batch)
kollab project file-status --file-id <returned-block-id>
# 3. Search (only once completed)
kollab project search --query "..." --project-id <project-id>
```

### Reset File Status (destructive)

Re-triggers the full processing pipeline from a target status. Use when a file is stuck or
after a pipeline bug fix. Requires `--confirm`.

```bash
# Default: reset to 'uploaded' and reprocess
kollab project reset-file-status --file-id <uuid> --confirm

# Reset to 'pending' without re-submitting
kollab project reset-file-status --file-id <uuid> --confirm --target-status pending --no-reprocess

# Reset but keep existing downstream artifacts
kollab project reset-file-status --file-id <uuid> --confirm --no-clear-artifacts
```

## Link Detail and Reprocess

```bash
# Get current details for a link block (processStatus, title, metadata, etc.)
kollab project get-link --link-id <uuid>

# Resubmit a link for content extraction (useful when initial processing failed)
kollab project reprocess-link --link-id <uuid>
```

Notes:

- `get-link` returns the same `BlockResponseDto` shape as `project items`, including
  `processStatus`, `spaceId`, `parentId`, and the link's `url`.
- `reprocess-link` re-runs the full transcript bridge pipeline — for social/media URLs this
  means re-fetching transcripts or re-scraping content. It does not skip completed stages.
- For file blocks that are stuck, use `reset-file-status` instead.

## Batch Children

Fetch direct children of multiple parent blocks in a single call. Avoids N calls to
`project items --id X` when you need to enumerate several projects or folders at once.

```bash
# Accepts CSV or JSON array; max 100 IDs per call
kollab project items-batch --parent-ids "uuid1,uuid2,uuid3"
kollab project items-batch --parent-ids '["uuid1","uuid2"]'
```

Returns `{ items: [{ parentId, children: BlockResponseDto[] }] }`. Children inaccessible to
the caller are silently omitted by the server. Use `block-permission-role` to confirm access
before batch-fetching if unexpected empty results appear.
## Project Memory

- Read: `kollab project get --id <project-id>`
- Set text: `kollab project update --id <project-id> --memory "<complete-memory>"`
- Set file: `kollab project update --id <project-id> --memory-file <utf8-path>`
- Clear: `kollab project update --id <project-id> --clear-memory`

Memory is a dedicated 8,000-character Project field, not a section inside `description`. Update writes read the current value and send an expected-value guard; on HTTP 409, reread and reconcile instead of retrying stale text.
