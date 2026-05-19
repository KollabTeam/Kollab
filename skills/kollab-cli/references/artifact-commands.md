# artifact commands

Artifact commands cover conversation-, project-, and space-level lookup plus common artifact actions.

## Querying

```bash
kollab artifact list --conversation-id <conversation-id> --include-workspace-sync
kollab artifact list --project-id <project-id>
kollab artifact list --space-id <space-id> --page 1 --page-size 20
kollab artifact url --id <artifact-id>
```

Notes:

- exactly one of `--conversation-id`, `--project-id`, or `--space-id` must be provided
- space-level listing supports pagination
- `includeWorkspaceSync` also returns workspace-sync artifacts

## Common Actions

```bash
kollab artifact save-to-space --id <artifact-id> --parent-block-id <block-id>
kollab artifact install-skill --id <artifact-id>
kollab artifact update-content --id <artifact-id> --content "new content"
kollab artifact delete --id <artifact-id>
```
