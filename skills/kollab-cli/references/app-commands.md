# app commands

Hosted Apps are the default publishing path for every new website and web app.
They publish user supplied `.html`, `.htm`, or `.zip` sources under the current
Workspace with a stable `*.kollab.host` identity. The CLI uses the authenticated
managed Kollab service.

## Credentials

`kollab app` accepts, in this order: a `kollab login` session, a host-injected workspace token
(`KOLLAB_API_TOKEN` / `KOLLAB_TOKEN`), or an account API key (`KOLLAB_API_KEY`). The API key is the
path for third-party agents (Claude Code, Codex, a custom agent) that must manage sites without a
browser login.

Get the key once, as the account owner, from a logged-in shell:

```bash
umask 077
kollab api-key create --name "site agent" --space-id <workspace-uuid> > ./kollab-key-response.json
# The raw key is printed once. Store it as the third party's secret; revoke with `kollab api-key revoke <id>`.
# Without --space-id the key is bound to the workspace of the current login (`space_id` in `kollab status`).
```

Give the third-party agent only these environment variables:

```bash
export KOLLAB_API_KEY="<one-time value>"          # required
kollab app list
kollab app deploy --file ./site.zip               # prints appId and url
kollab app update --id <app-id>                    # rebuild from the app's bound Task files
kollab app unpublish --id <app-id>
kollab app publish --id <app-id>
kollab app rollback --id <app-id>                 # undo the latest update/publish, if available
kollab app info --id <app-id>                     # status, and the `rollback` field
```

Scope of the key for `kollab app`: the CLI exchanges it for a 1-hour credential (never stored,
never printed) that the server limits to the key's default workspace (`--space-id` at creation, else
the login workspace at creation time; `KOLLAB_SPACE_ID` cannot widen it) and to Hosted App routes plus
the blank-task and upload calls a deploy needs; those task and upload calls are rejected (403) for any
other workspace, and the credential is refused by `/standalone/v1/*` (tools, jobs, models). Apps in other workspaces return an error, and other command groups (`task`,
`project`, `space`, ...) keep rejecting the key. The same key still works for `model`, `tool`,
and `social`. Revoking the key stops new exchanges; an already issued credential expires within an
hour. If a login session or `KOLLAB_API_TOKEN` is present, it wins and the key is not used.

## Package format

- A single `.html` / `.htm` file publishes as a static page.
- A `.zip` with `app.manifest.json` at its root follows the Hosted App package contract (static
  `public/index.html`, or a Python/Node backend; consult the current Hosted App documentation at https://kollab.im/cli).
- A `.zip` without a manifest is treated as a plain static site: `index.html` must be at the ZIP
  root or inside a single top-level folder (for example `catsite/index.html`); all files are
  served as static assets.

A package that fails validation returns the concrete reason (for example `ZIP needs
app.manifest.json, or index.html at its root ...` or `Remove embedded credentials before
publishing`). Fix the package and retry; it is not a platform outage. Only
`HOSTED_APP_RUNTIME_UNAVAILABLE` means the runtime itself could not be reached.

## Deploy and update

```bash
# Deploy creates a new stable app identity. With no conversation id it creates a blank task first.
kollab app deploy --file /path/to/landing-page.html --project-id <project-id>
kollab app deploy --file /path/to/site.zip --conversation-id <conversation-id>

# Update from the same saved Task files used by the Web management button.
kollab app update --id <app-id>
# Agent-only recovery when the package source Task cannot be inferred.
kollab app update --id <app-id> --conversation-id <conversation-id> --source-root <app-directory-or-index.html>
# Explicit package replacement remains available for externally built files.
kollab app update --id <app-id> --file /path/to/new-site.zip --conversation-id <conversation-id>
```

The file must be non-empty and no larger than 20 MB. `--conversation-id` and `--project-id` are alternatives: a project is used only when the CLI needs to create the blank source task. The blank task is created with `modelTier: "auto"` and no chat turn is sent. The upload is synced into a canonical artifact before the publish request; if artifact sync fails, stop and report the error instead of attempting to deploy an untracked object.

For apps built inside a Kollab Task, keep the editable source files and package
in that same Task and pass its `--conversation-id` on deploy. The server records
the source automatically; subsequent `kollab app update --id` and the Web Update
button call the same packaging endpoint. A blank CLI source Task containing only
an uploaded ZIP has no editable directory to infer; repair its source link only
after verifying the original Task and directory.

The server resolves the package hash from the canonical artifact and verifies that both the target app and the source task/artifact belong to the same Workspace. A failed update restores the prior deployment and source identity. A new upload to the same task does not update an existing app unless `kollab app update --id <app-id>` is used explicitly.

### Quota full (`HOSTED_APP_LIMIT_REACHED`)

`deploy` returns `409 HOSTED_APP_LIMIT_REACHED` when the Workspace's published-app slots are full.
Do not respond by proposing or executing `kollab app update` against an existing app that is
unrelated to the current task, and never dress that up as an easy option ("nobody uses this one",
"it's just yesterday's test artifact") — that is a guess about someone else's app, not a fact you
can verify. Run `kollab app list`, report the existing apps as-is (name, URL, and what they are for,
with no subjective "still needed" judgment), and let the user decide which one to unpublish or
whether to upgrade the plan. Only call `kollab app update --id <app-id>` on an existing app when the
user names that specific app themselves, unprompted — not when they pick it from options the agent
put in front of them.

## Undo the latest change (rollback)

```bash
kollab app rollback --id <app-id>
```

The server keeps exactly one previous version per app — the one that was live right before the
most recent successful `update`/`publish`. `rollback` swaps the app back to that version; running
it again swaps back to the version you just left, so it doubles as an undo for a bad rollback too.
It takes no `--to` and no file: there is no history list to pick from, only "current" and "the one
before it". A successful call returns the same app view as `kollab app info --id <app-id>`, and its
`rollback` field flips to reflect the new state.

**Routing rule for an agent**: when the user says something like "回到上一版" / "撤销刚才的发布"
/ "刚才的更新有问题，恢复原来的" / "undo that update" / "revert to the previous version", do not
rebuild anything and do not call `kollab app update` with an old file. First run `kollab app info
--id <app-id>` and read its `rollback` field:

- `available` — call `kollab app rollback --id <app-id>` directly. This is the whole fix; no
  re-upload, no new task, no new artifact.
- `none` — there is no previous version to return to (the app has never been updated, or its last
  update failed and never went live). Say so plainly; if the user has the earlier source file
  themselves, offer `kollab app update --id <app-id> --file <old-file>` as the only way back.
- `unsupported` — the current version is a persistent (Fargate) backend, or has AI enabled;
  neither supports rollback yet. Say so plainly and, if the user still has the previous source,
  offer `kollab app update` with that file as the alternative. Do not silently fall back to
  rebuilding from memory.

Rollback can fail with three `400` error codes, each already surfaced by the CLI as a hint string
(`kollab app rollback` prints `{ "hint": "..." }` alongside the raw `error`):

| Code | Meaning | What to do |
| --- | --- | --- |
| `HOSTED_APP_NO_PREVIOUS_VERSION` | Same as `rollback: 'none'` above | Explain there is nothing to roll back to; offer `update` with the old source if the user has it |
| `HOSTED_APP_ROLLBACK_UNSUPPORTED` | Same as `rollback: 'unsupported'` above | Explain the app type/AI state blocks rollback; offer `update` with the old source if the user has it |
| `HOSTED_APP_PUBLISH_IN_PROGRESS` | Another `deploy`/`update`/`publish`/`rollback` for this app is still running | Poll `kollab app info --id <app-id>` until it settles, then retry |

## Read and lifecycle operations

```bash
kollab app list
kollab app info --id <app-id>
kollab app export --id <app-id>
kollab app export-status --id <app-id> --job-id <job-id>
kollab app usage --id <app-id>
kollab app publish --id <app-id>
kollab app unpublish --id <app-id>
kollab app rollback --id <app-id>
kollab app ai --id <app-id> --action pause
kollab app ai --id <app-id> --action resume
kollab app ai --id <app-id> --action budget --daily-credit-limit 20 --monthly-credit-limit 500
```

`list`, `info`, `export-status`, and `usage` are read operations. Creating an export, `deploy`, `update`, `publish`, `unpublish`, `rollback`, and `ai` are write operations. When `allowed_cli_commands` contains `app:read`, use only those read commands and domain read commands; `app` grants the complete app command group. The CLI check is only an early boundary: the server still validates the workspace token against the actual app and source resource on every request.

Every app view — the object `info` returns, and each entry in `list`'s `apps[]` — carries a
`rollback: 'available' | 'none' | 'unsupported'` field. Check it before deciding whether rollback
is even worth attempting; see "Undo the latest change (rollback)" above for the full routing rule.


A first backend deployment may return `status: "provisioning"` while private networking starts. Keep its app ID and check `kollab app info --id <app-id>` until published or failed; do not create another app to retry a pending deployment. A successful CLI HTTP response alone does not prove the public page is ready.

## Custom domain

Custom domains for new sites stay on the same Hosted App identity:

```bash
kollab app domain list --id <app-id>
kollab app domain status --id <app-id>
kollab app domain add --id <app-id> --domain www.example.com
kollab app domain verify --id <app-id>
kollab app domain remove --id <app-id>
```

Use the DNS records returned by `domain add`, then wait for `domainStatus:
active` before treating the custom hostname as ready.
