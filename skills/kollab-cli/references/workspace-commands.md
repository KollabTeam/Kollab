# workspace commands

`workspace` is the standard CLI namespace for workspace resource operations.

## Usage

Use these unified entrypoints:

```bash
kollab workspace space ...
kollab workspace project ...
kollab workspace task ...
kollab workspace skill ...
kollab workspace artifact ...
kollab workspace timer ...
kollab workspace bot ...
kollab workspace mcp ...
kollab workspace memory ...
```

## Examples

```bash
kollab workspace project list
kollab workspace project search --query "permission model"
kollab workspace task list
kollab workspace task create --project-id <project-id> --model-tier lite
kollab workspace task ask --question "Summarize this project" --project-id <project-id>
kollab workspace task answer-question --conversation-id <conversation-id> --tool-use-id <tool-use-id> --answers '{"approval":"allow"}'
kollab workspace artifact list --conversation-id <conversation-id>
kollab workspace bot list
kollab workspace memory list
```

## Notes

- this is the unified namespace for the existing CLI
- it directly reuses the current `space/project/task/...` command groups underneath
- the goal is to converge all workspace operations on one CLI surface
- when the runtime allows only part of the CLI, use the explicitly authorized `kollab workspace <group> ...` group from the system prompt
- `workspace task create` is the same WebUI-equivalent `/qa/conversation` flow as `task create`
- `workspace task ask` and `workspace task answer-question` return the same high-fidelity chat bundle as their `task ...` aliases
