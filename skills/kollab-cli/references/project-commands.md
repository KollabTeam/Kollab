# project commands

Project commands cover project CRUD, document search and reading, block operations, and project collaboration permissions.

## Basic CRUD

```bash
kollab project list
kollab project items --id <project-id>
kollab project create --title "Docs"
kollab project update --id <project-id> --title "New Name"
kollab project delete --id <project-id> --confirm
```

## Documents and Search

```bash
kollab project documents --project-ids '["<project-id>"]' --page 1 --page-size 50
kollab project read --file-id <file-id> --page 1 --page-size 5000
kollab project search --query "session storage" --project-ids '["<project-id>"]' --expand-nodes 1 --score-threshold 0.45
```

Notes:

- `--project-ids` and `--file-ids` accept comma-separated values or JSON array strings
- `search` already supports `expand-nodes` and `score-threshold`
- for project details, use `items`, `documents`, `read`, or the access endpoints

## Block Operations

```bash
kollab project create-link --project-id <project-id> --title "Spec" --url https://example.com
kollab project create-folder --project-id <project-id> --title "Notes"
kollab project update-block --block-id <block-id> --title "Renamed"
kollab project delete-file --block-id <block-id>
kollab project file-url --file-id <file-id>
```

## Collaboration Permissions

```bash
kollab project access --id <project-id>
kollab project invite --id <project-id> --email user@example.com
kollab project invite-link --id <project-id>
kollab project remove-member --id <project-id> --user-id <user-id>
```
