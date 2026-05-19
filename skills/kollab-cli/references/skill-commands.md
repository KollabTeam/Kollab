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

- hydrates one official skill into the current conversation's workspace via the backend endpoint `POST /tools/skills/hydrate-from-catalog`
- **session-scoped**: the skill is added only to this conversation's workspace; the space ACL, other conversations, and other users stay as they are
- effect persists across turns within the same conversation via workspace S3 sync, and stays local to that conversation
- after `try` succeeds, read the hydrated `SKILL.md` in `skills_root/<name>/SKILL.md` and follow its instructions
- `--expiration` is optional; omit it for conversation-lifetime scope

## Space-wide Permanent Install

```bash
kollab skill install --id <skill-id>
kollab skill install --github-url https://github.com/owner/repo/tree/main/skills/<skill-name>
```

Notes:

- adds the skill to the entire space permanently; affects all members and persists across sessions
- **always ask the user first** via `AskUserQuestion` before running — this mutates the space ACL for everyone
- `--id` installs an official catalog skill by ID
- `--github-url` imports a GitHub skill directory through skills-server market install, then saves the repository root as source attribution when SKILL.md has not provided richer metadata
- `--id` and `--github-url` are mutually exclusive
- the current CLI covers official install and GitHub import; custom skill update/delete is managed elsewhere
