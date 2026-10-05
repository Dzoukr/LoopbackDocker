# Loopback (Docker)

Run [Loopback](https://github.com/Dzoukr/Loopback) from prebuilt images: it syncs your Plaud recordings, transcribes them with Speechmatics and turns them into structured notes with your local `claude` CLI. Nothing to build - this folder is all you need.

## Requirements (Windows)

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) running
- [.NET SDK 10](https://dotnet.microsoft.com/download) - runs the Claude bridge (`dotnet fsi`)
- [Claude Code](https://claude.com/claude-code) CLI installed and logged in (`claude` on PATH)
- A Plaud account with a password (no 2FA) and a Speechmatics API key

## Setup (once)

Copy `.env.example` to `.env` and fill in the required values. `CLAUDE_BRIDGE_TOKEN` is any long random string.
Optionally set `LOOPBACK_OUTPUT` to where the result files should go (e.g. your Obsidian inbox).

## Run

| | |
|---|---|
| `loopback-up.cmd` | Start - or update to the newest images. UI: http://localhost:3000 |
| `loopback-down.cmd` | Stop everything (your data is kept) |

`loopback-up.cmd` also starts the Claude bridge (`claude-bridge.fsx`) on the host and registers it to start at logon - a container cannot use your `claude` login, so it runs outside Docker. Its log is `claude-bridge.log`.

To pin a version instead of `latest`, set `LOOPBACK_VERSION=1.4.0` in `.env`.

## Workflows

A workflow decides how a recording is processed. Each subfolder of `workflows/` is one workflow (the folder name appears in the UI) with three prompts: `summary.md`, `merge.md`, `title.md`. Optionally add:

- `schema.json` - shape of the result (default: `workflows/default.schema.json` - summary, key points, action items)
- `claude.json` - override `model`, `effort`, `passes` or `timeoutSeconds` from `workflows/default.claude.json`

Copy `workflows/Default` to start a new one. Changes are picked up without a restart.

## Data

- Result JSON files: `LOOPBACK_OUTPUT` (default `./output`)
- Database (incl. the Plaud session): Docker volume `loopback-data`. Back it up with `docker compose cp server:/data/loopback.db .`
- Remove everything incl. the database: `docker compose down -v`
