# auth commands

Use these commands to manage the CLI login state.

## `kollab login`

Sign in and persist the token / API URL locally.

By default, this opens a browser URL, lets the user log in with the normal web account, and asks them to choose one space for the CLI session.

```bash
kollab login --api-url https://kollab.im
kollab login --token <token> --api-url https://kollab.im
```

Parameters:

- `--token <token>`: optional; if provided the CLI skips the browser flow and validates this token directly
- `--api-url <url>`: optional; use the site base URL, not `/api`

Notes:

- The browser flow is now the default and the recommended path for humans.
- `--token` is the fallback for scripts, tests, or pre-issued workspace tokens.

## `kollab logout`

Clear locally stored credentials.

```bash
kollab logout
```

## `kollab status`

Inspect whether the CLI is currently authenticated, and which API URL, user, and space binding are active.

```bash
kollab status
```
