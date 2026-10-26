# user commands

User commands read and update the authenticated account's profile and preferences.
These are account-level operations, not space/workspace resource operations — no spaceId is required.

## Profile

```bash
# Get the current authenticated user's full profile
kollab user profile
```

Returns a `UserDto` with fields including `uuid`, `displayName`, `avatarUrl`, `email`, `preferences`, and `createdAt`.

## Update Profile

```bash
# Update display name only
kollab user update --name "Alice"

# Update display name and avatar URL
kollab user update --name "Alice" --avatar-url "https://example.com/avatar.png"
```

Notes:

- `--name` is required even when only `--avatar-url` is changing (server DTO constraint)
- display name: 1–50 characters
- avatar URL: max 500 characters

## Preferences

```bash
# Read current preferences
kollab user preferences
```

Returns the full user profile (preferences live inside `UserDto`). The preferences field is a free-form JSON object.

## Admin Check

```bash
# Check whether the current user has system admin permission
kollab user is-admin
```

Returns `{ isAdmin: boolean }`. `isAdmin: true` means the caller has Casdoor-level system admin,
not just space-level admin. Space admin is separate and resolved by `kollab space info`.

## Set Preferences

```bash
# Replace all preferences with a full JSON object
kollab user set-preference --json '{"theme":"dark","language":"en","accentColor":"coral"}'

# Set a single key (reads current preferences and merges the key before writing)
kollab user set-preference --key theme --value dark
kollab user set-preference --key bentoLayout --value true
kollab user set-preference --key maxTokens --value 4096
```

Notes:

- `--json` replaces the entire preferences object; use it when changing multiple keys at once
- `--key / --value` is a convenience wrapper: it fetches the current preferences and merges the single key before writing. A concurrent update between read and write could silently drop one change — use `--json` for safety when making multiple changes
- `--value` is automatically parsed as JSON (`true`/`false`, numbers, objects); plain strings that are not valid JSON are passed through as strings
- the total preferences object must not exceed 4096 bytes (server-enforced)
