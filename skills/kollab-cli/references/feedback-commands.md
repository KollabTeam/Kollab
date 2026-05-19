# feedback commands

`kollab feedback` is how an agent triages and files issues against the
public `KollabTeam/Kollab` repository. It is the sanctioned channel
for surfacing product-level problems hit during a task; always route
product feedback through this command.

The agent always **searches first**, then decides whether to comment on
an existing issue or file a new one. Deduping is the agent's
responsibility — the search step is what catches duplicates.

## Standard flow

```
user describes a problem
  -> kollab feedback search -q "<1-3 keywords>" --state all
  -> read returned issues (look at title, body, state):
     - closed issue with a clear fix: tell the user "this is fixed in vX, please update"
     - open issue describing same problem: kollab feedback comment --issue <n> -m "<your repro>"
     - no relevant matches: kollab feedback report -m "<your repro>"
     - ambiguous: ask the user before report / comment
```

Search is cheap. Always run it before `report` or `comment` so each filed
issue is unique and worth a maintainer's attention.

## search

```bash
kollab feedback search -q "drag space sidebar reorder" --state all --limit 5
```

Options:

- `-q, --query <text>` (required): pick **1-3 meaningful terms** in
  English or the user's language. The server strips GitHub Search
  qualifiers (anything containing `:`, `<`, `>`, `"`) before sending,
  so plain keywords always stay scoped to the target repo even if a
  phrase like `repo:other/repo` or `is:wontfix` slips in.
- `--state <open|closed|all>` (default `all`):
  - `open` only: see what is currently being tracked
  - `closed` only: see if this was already fixed
  - `all` (recommended for triage): see both, decide based on what comes back
- `--limit <n>` (default 5, max 10)

Returns:

```json
{
  "ok": true,
  "data": {
    "issues": [
      {
        "number": 42,
        "title": "Sidebar drag stops responding after collapse",
        "body": "...",
        "url": "https://github.com/KollabTeam/Kollab/issues/42",
        "state": "open"
      }
    ]
  }
}
```

If no issues match: `issues` is `[]`.

## report

```bash
kollab feedback report -m "When I drag a space to the bottom of the sidebar, it snaps back. Repro: drag any space to position 0, release."
```

Options:

- `-m, --message <text>` (required, 5-5000 chars): write a focused
  reproduction in the user's own words and language. Keep it under a
  paragraph; long noise gets stripped less reliably.

The CLI auto-attaches:

- `taskUrl`: derived from `creds.api_url` + `$CURRENT_CONVERSATION_ID`
- `conversationId`: `$CURRENT_CONVERSATION_ID`
- `runtimeId`: `$AGENTCORE_RUNTIME_ID` / `$BEDROCK_AGENTCORE_RUNTIME_ID`
- `locale`: `$KOLLAB_LOCALE` / `$LANG`

Returns the new `issueUrl` and `issueNumber`. Show the URL to the user.

## comment

```bash
kollab feedback comment --issue 42 -m "Hit this on Safari iOS 17 too; same drag-to-bottom snapback."
```

Options:

- `--issue <number>` (required): use an issue number returned by a
  `feedback search` result so it resolves in `KollabTeam/Kollab`; the
  server's GitHub App is scoped to that one repo.
- `-m, --message <text>` (required, 5-5000 chars): additional repro,
  reproduction environment, "I hit this too" signal, etc. The CLI
  attaches the same task / conversation / runtime context as `report`.

Returns the new `commentUrl`.

## Capability scope

- The available operations are search, report, and comment. The GitHub
  App is scoped to `Issues: Read & Write`, so stick to those three verbs.
- All operations target the single repo `KollabTeam/Kollab`; the server
  hardcodes the `repo:` qualifier on search and the App is installed
  only on that repo.
- All written text passes through a server-side redactor that strips
  emails, IPs, internal subdomains, long hex strings, and token-like
  prefixes (`ghp_`, `Bearer ...`). Write plainly and let the redactor
  handle sensitive fragments.

## Permissions

`feedback` is in the CLI's `UNRESTRICTED_COMMANDS` allowlist
(`packages/kollab-cli/src/auth/cli-restrictions.ts`), alongside
`login` / `logout` / `status` / `help`. This means:

- A bot whose `allowed_cli_commands` is set to `[]` (CLI fully disabled
  for workspace operations) can **still** run `kollab feedback ...`.
- A bot whose `allowed_cli_commands` is `["task"]` (only task ops) can
  also still run `kollab feedback ...`.
- The server-side endpoints under `/api/feedback*` do their own auth
  check using the workspace token; permission is never delegated to
  whatever `allowed_cli_commands` happens to be.

Reasoning: feedback is about Kollab the product, not about a specific
workspace resource. Keeping it always-available ensures the people
hitting bugs can always report them, regardless of CLI restrictions.

## Common failures

- `503 GitHub Feedback App is not configured`: env keys
  `GITHUB_FEEDBACK_APP_ID` / `GITHUB_FEEDBACK_INSTALLATION_ID` /
  `GITHUB_FEEDBACK_PRIVATE_KEY` are missing on the server. Ask the
  platform owner to configure them, and treat this as terminal for now.
- `400 description is too short` / `message is too short`: the body
  must be at least 5 characters.
- `400 description is too long` / `message is too long`: keep it under
  5000 characters; put large reproductions in a follow-up comment after
  the first submission.
- `400 query is empty after sanitization`: the search query was
  stripped to nothing because every token contained a forbidden
  character (`:`, `<`, `>`, `"`) or started with `[`. Pick simpler
  English / source-language keywords.
- `404` from `feedback comment`: that issue number does not exist
  in `KollabTeam/Kollab`. Re-run `feedback search`, pick a real number.

For deeper diagnosis (PEM rotation, App reinstall, audit log lookup),
see `$debug-kollab` `troubleshooting.md` under the CLI feedback incident.
