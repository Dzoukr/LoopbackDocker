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

`loopback-up.cmd` also starts the Claude bridge (`claude-bridge.fsx`) on the host and registers it to start at logon - a container cannot use your `claude` login, so it runs outside Docker. Its log is `claude-bridge.log`. When `claude-bridge.fsx` was updated (e.g. after `git pull`), `loopback-up.cmd` restarts the running bridge.

To pin a version instead of `latest`, set `LOOPBACK_VERSION=1.4.0` in `.env`.

## Marks and attachments

- Drop files (screenshots, documents) on a recording - or paste with Ctrl+V - to attach them. Their time in the call comes from the file name, EXIF or the file date, and can be edited.
- Marks made in Plaud during the call (device button highlights, text notes, voice memos, photos) are imported during sync. Voice memos are transcribed by Speechmatics.
- Processing reads them next to what was said at that moment: Claude describes images / PDFs first (`describeModel`, default haiku), and the result assigns each one to a topic.
- The result file (`"format": 2`) lists `marks` and `attachments`; the attachment files are copied next to it into `loopback-<id>.files/`.

## Workflows

A workflow decides how a recording is processed. Each subfolder of `workflows/` is one workflow (the folder name appears in the UI) with three prompts: `summary.md`, `merge.md`, `title.md`. Optionally add:

- `schema.json` - shape of the result (default: `workflows/default.schema.json` - summary, key points per topic, decisions, open questions, action items with owner and due date, and the topic of each note / attachment)
- `claude.json` - override `model`, `effort`, `passes`, `timeoutSeconds`, `describeModel` or `describeEffort` from `workflows/default.claude.json`

`workflows/describe.md` (optional) replaces the built-in prompt for describing attachments.

Copy `workflows/Default` to start a new one. Changes are picked up without a restart.

## Data

- Result JSON files (and their `loopback-<id>.files/` folders): `LOOPBACK_OUTPUT` (default `./output`)
- Database (incl. the Plaud session): Docker volume `loopback-data`. Back it up with `docker compose cp server:/data/loopback.db .`
- Remove everything incl. the database: `docker compose down -v`
