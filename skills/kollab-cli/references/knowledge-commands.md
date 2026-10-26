# knowledge commands

`knowledge` is the standard CLI entrypoint for knowledge-base Q&A and document retrieval.

## Commands

- `kollab knowledge ask`
- `kollab knowledge search`
- `kollab knowledge keyword-search`
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

kollab knowledge keyword-search \
  --question "exact term, name, id, or code to find" \
  --project-id <project-id>

# Narrow retrieval to specific files inside the project
kollab knowledge ask \
  --question "What does the spec say about retries?" \
  --project-id <project-id> \
  --file-ids <file-uuid-1>,<file-uuid-2>

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
- `ask` / `search` accept `--file-ids` (comma-separated or JSON array) to scope retrieval to specific files inside the project; `keyword-search` has no such filter
- `ask` / `search` / `keyword-search` accept the search text as `--question` or its alias `--query` (either works). Note: on `list-documents`, `--query` is a different option — a title/filename filter, not the search text.
- `keyword-search` runs a BM25 sparse (lexical) lookup via `/qa/knowledge/keyword-search` and returns the raw matching chunks (`documentId / title / snippet / score`), NOT an LLM answer. Use it for exact term / name / id / code-token lookups that the semantic `ask`/`search` underweights — generate the precise keywords and search them directly, then read the snippets. (`ask`/`search` already fuse semantic + keyword automatically; reach for `keyword-search` when you want a pure, explainable lexical match.) The bm42 encoder tokenizes Latin/Cyrillic words + CJK/kana/Hangul bigrams (with full-width normalization), so keyword-search works across EN/ZH/JA/KO/RU.
- `list-documents` returns stable fields such as `document_id / project_id / file_type / updated_at`
- `get-document-content` returns stable fields such as `document_id / page / page_size / pagination / content`
- these commands do not go through chat stream, so they do not create task runtime events
