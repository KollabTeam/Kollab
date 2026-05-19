# knowledge commands

`knowledge` is the standard CLI entrypoint for knowledge-base Q&A and document retrieval.

## Commands

- `kollab knowledge ask`
- `kollab knowledge search`
- `kollab knowledge list-documents`
- `kollab knowledge get-document-content`

## Examples

```bash
kollab knowledge ask \
  --question "Summarize the core goal of this project" \
  --project-id <project-id>

kollab knowledge search \
  --question "What are the permission constraints in this requirement doc?" \
  --project-id <project-id> \
  --top-k 8

kollab knowledge list-documents \
  --project-id <project-id> \
  --query PRD

kollab knowledge get-document-content \
  --document-id <file-uuid> \
  --page 1 \
  --page-size 3000
```

## Semantics

- `ask` and `search` both hit `/qa/knowledge/ask` directly and return a concise retrieved answer
- `search` does not use a different backend; it just makes a search-first intent more explicit
- `--top-k` is an optional retrieval-depth hint; the backend knowledge agent still decides the final recall strategy
- `list-documents` returns stable fields such as `document_id / project_id / file_type / updated_at`
- `get-document-content` returns stable fields such as `document_id / page / page_size / pagination / content`
- these commands do not go through chat stream, so they do not create task runtime events
