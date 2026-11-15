# Knowtion

An open-source, local-first knowledge workspace — nested pages, block documents,
rich text, and databases with typed properties and views.

**Your notes live on your machine.** Sync is optional and goes through storage you
already own. There is no hosted Knowtion account or subscription. You can also run the
optional single-user server on your own machine.

> Status: **pre-alpha, and runnable.** Nested pages, the block editor, full-text search,
> multi-device sync through a shared folder, end-to-end encryption, and databases — eight
> property types, table and board views, filter, sort and group, drag ordering — all work
> today, and a Notion export imports with its databases. See the roadmap below.
>
> **Desktop app: Windows.** Any OS can run the self-hosted web build — see _Self-host it_.

## Install it

Windows desktop builds are planned. This repository does not yet have a published
installer; see _Development_ to run the app from source, or _Self-host it_ for web.

Windows will show a blue "Windows protected your PC" box, because the installer is not
signed by a certificate Microsoft recognises. Click **More info**, then **Run anyway**.
Removing that warning means buying a code-signing certificate, which this project has not
done. The installer needs no administrator password, and uninstalling leaves your notes
where they are.

Knowtion tells you when a newer version exists and links you to it; it never downloads or
installs anything by itself. That check is the only request it makes on its own, it sends
nothing about you, and you can turn it off under Settings. See
[RELEASING.md](RELEASING.md) for why it works that way.

**It is pre-alpha.** Keep a copy of anything you cannot lose. If you are testing it
deliberately, [docs/TESTING.md](docs/TESTING.md) is a checklist of everything it can do.

## Self-host it (web)

The same engine runs as a web app on any machine with Docker. One shared password, one
workspace; the browser is a view onto it.

    KNOWTION_PASSWORD='something long' docker compose up -d   # http://localhost:3000

Or without Docker: `KNOWTION_PASSWORD=... npm run serve -w @knowtion/desktop`.

- **Data** lives in the `/data` volume (`KNOWTION_DATA`). Back it up; it holds the
  encrypted log _and_ the key to it, so treat the volume as secret. The recovery phrase
  ceremony still runs on first visit.
- If the local key file is lost but the original identity and workspace log are still
  present, Knowtion asks for the saved recovery phrase at startup and restores the saved
  key epochs. The phrase alone cannot recreate a deleted workspace or identify its log.
- **HTTPS**: put it behind a reverse proxy and set `KNOWTION_SECURE_COOKIE=1`.
- **Not in the web build**: sync folders and device approval (the server is the store),
  update checks. Export writes to `/data/exports/`; import is a zip upload.
- **Live tabs**: multiple online browser tabs merge edits and refresh after reconnecting.
  Offline edits in the web app are not saved; use the desktop app for offline work.

### Ask your notes with a local model

The assistant is optional and off by default. Install Ollama on your server, download a
local model, and start Knowtion with `KNOWTION_AI_MODEL` set to that model's name. For
the Docker setup, Knowtion connects to Ollama on the host at
`http://host.docker.internal:11434` by default; set `KNOWTION_AI_URL` if your Ollama
server is somewhere else. The model server must accept connections from the Knowtion
container; keep its API on a private interface and firewall it from the public internet.
Restart the container after changing these settings.

Choose **Ask your notes**, enter a question, and open the listed pages to check the
answer. Knowtion sends matching page text and the question to the configured model only
when you ask. Use a locally installed model and a server you trust: a remote endpoint or
cloud model may send your notes outside your machine. Answers can be wrong, and search
may miss relevant pages. The assistant currently searches page titles and text, not
attachments or every database property.

### Back up and restore

Stop the app before copying its volume so pending edits are flushed and the backup is
consistent. Run these commands from the project directory, using the same Compose
project and password as your installation:

    docker compose stop
    umask 077
    docker compose run --rm --no-deps --entrypoint tar knowtion -czf - -C /data . > knowtion-backup.tar.gz
    docker compose start

The archive contains both your notes and their keys. Store it somewhere private and
keep an additional copy away from the server. If backup fails, start the app again with
`docker compose start` and investigate before relying on the archive.

To test a restore without overwriting your workspace, use a separate Compose project
and volume. Stop it before importing the backup:

    docker compose -p knowtion-restore run --rm --no-deps --entrypoint tar knowtion -xzf - -C /data < knowtion-backup.tar.gz
    docker compose -p knowtion-restore run --rm --no-deps -p 3001:3000 knowtion

Open `http://localhost:3001`, sign in with the configured password, and check your
pages. Stop the test with Ctrl+C, then remove its temporary volume with
`docker compose -p knowtion-restore down -v`. Use a fresh project name for each test;
never extract a backup over a running or existing workspace.

## Why another Notion alternative

Most alternatives ask you to trust their cloud, or give up databases and views to get
local files. Knowtion aims for both:

- **Local-first desktop app.** It edits offline; the network is never in the path of a
  keystroke. The self-hosted web app needs a connection to your own server to load and
  save changes.
- **No hosted account.** The desktop app needs no account and never asks for access to
  your cloud account. You point it at a folder your existing Google Drive, OneDrive,
  Dropbox or Syncthing client syncs, and that client does the transport. The web build is
  optional and runs on your server.
- **Encrypted shared-folder sync.** Desktop sync stores ciphertext in the folder your
  cloud provider sees. A workspace will not open until you have written down its recovery
  phrase, because an append-only log cannot un-write a note saved in the clear.
- **Your data stays yours.** Export the whole workspace to Markdown or JSON whenever you
  like, no lock-in. The Markdown is a folder of files any editor opens; the JSON holds
  everything, addressed by identifiers that mean something outside Knowtion.

## Roadmap

The goal is Notion parity: every Notion feature, free and open source, working offline
with no account. [docs/ROADMAP.md](docs/ROADMAP.md) lists every gap and the order they
close in. Next up is making the editor usable without memorising shortcuts, then images
and files.

## Platform support

**Windows only at v1.** The engine is portable and the test suite runs on Linux in CI,
but there are no macOS or Linux releases yet.

Folder mode relies on a desktop sync client watching the folder you choose, and neither
Google nor Microsoft ships one for Linux — Linux users would need Dropbox, Syncthing,
rclone or insync. macOS additionally needs Apple Developer enrolment before it can
auto-update, since Squirrel.Mac refuses to update an unsigned app. Both are deferred
rather than ruled out.

## Development

Requires Node 24+.

    git clone <repo-url>
    cd knowtion
    npm install
    npm run verify     # formatting, typecheck, lint, licence gate, tests
    npm run format     # rewrite anything the formatting check objects to
    npm start          # build and launch the desktop app

If `npm install` skips lifecycle scripts in your environment, Electron's binary will be
missing. Fetch it with:

    node node_modules/electron/install.js

Your notes live in the OS application-data directory under `Knowtion/log`, as immutable
packs. Nothing is written for a workspace you have not edited.

See CONTRIBUTING.md before opening a pull request — it contains a **mandatory
clean-room rule** about copyleft prior art.

## Documentation

| Document        | Purpose                                         |
| --------------- | ----------------------------------------------- |
| ARCHITECTURE.md | How the system fits together (5 pages, no more) |
| FORMAT.md       | The on-disk and on-cloud format specification   |
| docs/ROADMAP.md | The Notion-parity gap list and its order        |
| docs/adr/       | Architectural decision records — append-only    |
| THIRD_PARTY.md  | Dependency licences (generated by CI)           |
| SECURITY.md     | Threat model and vulnerability disclosure       |

## Licence

MIT — see LICENSE.

Knowtion is not affiliated with, endorsed by, or derived from Notion Labs, Inc.
