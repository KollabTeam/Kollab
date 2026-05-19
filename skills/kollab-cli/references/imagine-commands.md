# imagine commands

`kollab imagine` is the public, read-only entry into the Kollab Prompt
Gallery, the same data that powers the `https://kollab.im/gpt-image-2-prompts`
landing page. It is the recommended way for any agent or terminal user to
**find a community-ranked prompt for a styled image / video request** before
calling `gptimg generate`, `seedance2 generate`, or `klingv3 generate`.

The gallery is populated from a curated Notion database, synced into the
`landing_prompt*` Postgres tables, and exposed by `apps/landing-pages`.
Results are pre-sorted by `like_count DESC, updated_at DESC` server-side;
**trust the server order** — the first item is already the best match.

## When to use this command group

- The user asks for any styled visual: editorial / fashion / luxury / product
  / sneaker drop / hanfu / kimono / anime / poster / infographic / dream /
  fairy tale / etc.
- Before writing a multi-paragraph prompt by hand: run
  `kollab imagine search` first, since the gallery probably already has a
  better community-tuned version
- The user says "show me a few popular prompts for X" or "what is hot for X right now"

When the user gives a fully-formed prompt of their own that they
explicitly want generated as-is, generate it directly.

## search

```bash
kollab imagine search -q "editorial" --tag fashion_beauty --lan zh-CN --limit 3
```

Options:

- `-q, --query <text>`: free-text search across `title` and all-language
  `prompts` JSONB; case-insensitive substring match. Keep it 1-3 words.
- `-t, --tag <tag>`: one of the 15 canonical tags (see `tags` below). Tag
  is the strongest filter; prefer tag over a long `q`.
- `-l, --lan <code>`: output language for the `prompt` field. Supported:
  `en`, `zh-CN`, `zh-TW`, `ja`, `ko`, `fr`, `de`, `ru`, `pt-BR`. Defaults
  to the detected locale (`KOLLAB_LOCALE` / `LANG`), falling back to `en`.
- `--limit <n>`: max items to return, 1-10. Defaults to 10.
- `--cursor <c>`: opaque pagination cursor from a previous response.
  Advanced; rarely needed because the first page is already sorted by
  community likes.
- `--top`: print only `items[0].prompt` as raw text (no JSON envelope).
  Use this when piping into a generator: `gptimg generate "$(kollab imagine
  search -q hanfu --top)" --size 1024x1536`.

Always set at least one of `-q` or `-t`; the server requires a filter so
each search stays scoped to a real query or tag.

### Default output

```json
{
  "ok": true,
  "data": {
    "query": { "q": "editorial", "tag": "fashion_beauty", "lan": "zh-CN", "limit": 3 },
    "items": [
      {
        "id": "uuid",
        "title": "@author - source-id",
        "prompt": "the actual prompt text in the requested language",
        "notionCreatedAt": "2026-04-15T08:00:00Z",
        "media": [
          {
            "id": "asset-uuid",
            "type": "image",
            "url": "https://cdn.kollab.im/.../original.jpg",
            "cardUrl": "https://cdn.kollab.im/.../thumb_768_q80.webp",
            "detailUrl": "https://cdn.kollab.im/.../original.jpg",
            "width": 1024, "height": 1536, "durationMs": null
          }
        ],
        "imageUrl": "https://cdn.kollab.im/.../original.jpg",
        "authorName": "BubbleBrain",
        "authorUrl": "https://x.com/BubbleBrain",
        "sourceUrl": "https://x.com/BubbleBrain/status/...",
        "tags": ["portrait_person", "fashion_beauty", "..."],
        "likeCount": 364,
        "accentIndex": 0
      }
    ],
    "hasMore": true,
    "nextCursor": "db:eyJvZmZzZXQiOjEwfQ",
    "availableTags": ["photography_cinematic", "..."]
  }
}
```

### --top output

```text
A model in <full prompt as authored> ...
```

No JSON wrapper, no trailing newline. Designed for direct
shell substitution.

## tags

```bash
kollab imagine tags
```

Returns the canonical tag set the gallery currently exposes. The set is
fixed at 15 entries:

```
photography_cinematic   portrait_person          fashion_beauty
product_brand_ad        architecture_city        anime_character
typography_text         social_ui_screenshot     storyboard_sequence
infographic_education   fantasy_sci_fi           object_design
reference_edit          food_drink               vehicle_transport
```

The special pseudo-tag `lasted` switches ordering to recency
(`notion_created_at DESC`) and is only useful when the user explicitly
asks for "newest". It is not returned by `imagine tags` because it is not
a content category.

## Failure handling

- `No prompts matched.`: drop `-q`, broaden the tag, or run `imagine tags`
  to pick a closer category.
- `Gallery API HTTP 502 ...`: sync delay or DB hiccup. Retry once. If it
  persists, the skill body has a fallback path that builds a prompt from
  the seven-slot anatomy.
- `Gallery API timed out after 15000ms`: network latency. Retry once.
- All errors print the standard CLI envelope `{ ok: false, error, ... }`
  to stderr and exit non-zero, matching every other Kollab CLI command.

## Notes

- Read-only public endpoint; no Kollab credentials needed.
- The Notion-sync path (`/api/internal/landing-prompts/...`) is service-only
  and not exposed through this CLI.
