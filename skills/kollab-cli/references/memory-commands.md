# memory commands

Memory commands read and maintain workspace memory files.

## List and Read

```bash
kollab memory list
kollab memory get --name MEMORY.md
```

Notes:

- the current CLI automatically uses the authenticated `space_id`, so `--space-id` is handled for you
- `name` is URL-encoded at the request layer

## Save and Delete

```bash
kollab memory save --name engineering-notes.md --content "latest notes"

kollab memory delete --name engineering-notes.md
```

Notes:

- `save` replaces the full file contents with what you pass
- prefer explicit `.md` filenames
