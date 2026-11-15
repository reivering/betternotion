# BetterNotion

BetterNotion is a private, self-hosted workspace setup based on AFFiNE. This repository holds the deployment configuration and personal defaults; the app itself runs from AFFiNE's official Community Edition container.

The self-hosted server is labeled **BetterNotion**. The AFFiNE client UI keeps its upstream branding. AFFiNE source is linked as a Git submodule at `upstream/AFFiNE`; it remains under its upstream license and is not copied into this repository's history.

Clone the source link when you need to inspect or build the upstream app:

```sh
git clone --recurse-submodules https://github.com/reivering/betternotion.git
```

The submodule is pinned to the reviewed upstream commit. Updating it is a separate change from updating the server image.

## Start your private server

You need Docker Engine and Docker Compose v2. From this directory:

```sh
cp .env.example .env
```

Edit `.env` and set `DB_PASSWORD` to a long random hexadecimal value. Generate one with `openssl rand -hex 32`. Compose refuses to start while it is blank. Then prepare the persistent directories and configuration:

```sh
mkdir -p config data/postgres data/storage
cp config/config.json.example config/config.json
```

Keep the default `server.externalUrl` for a local test. Start the stack:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

Open [http://localhost:3010/admin](http://localhost:3010/admin) and create the first administrator while the app is bound to localhost. After setup, disable public signups in the admin settings before exposing the server through a domain or reverse proxy. For a public domain, set `server.externalUrl` in `config/config.json` to its HTTPS URL and follow AFFiNE's HTTPS and reverse-proxy guidance.

The app binds to `127.0.0.1` by default. Keep that setting when a reverse proxy runs on the same server. If you need direct LAN access, change the host binding in `docker-compose.yml` and firewall the port to your trusted network.

## Data and backups

- `data/postgres` stores pages, accounts, and workspace metadata.
- `data/storage` stores uploaded workspace files.
- `config` stores the server config.
- `.env` contains the database password. Keep it private and back it up securely.

Back up all four together. Use the procedure in [docs/backup-restore.md](docs/backup-restore.md), and test restores before upgrading. Do not delete these paths when updating the app.

## Updates

The image tag defaults to `stable`. Pulling a newer image can run database migrations. Read the upstream upgrade guide, make a tested backup, then update during a maintenance window. For repeatable deployments, set `AFFINE_IMAGE_TAG` in `.env` to a specific AFFiNE release tag instead of `stable`.

## Upstream and architecture

AFFiNE provides the editor, workspace model, sync, and server. Its upstream repository is [toeverything/AFFiNE](https://github.com/toeverything/AFFiNE), and its architecture and build notes live in the linked source. The self-host stack runs the AFFiNE app, PostgreSQL, and Redis; PostgreSQL and uploaded files persist under `data/`, while Redis remains internal to the Compose network.

This project customizes the self-host deployment and server name. It does not claim feature parity beyond what the selected AFFiNE release provides.

## License

The deployment files and documentation authored in this repository are MIT-licensed; see [LICENSE](LICENSE). AFFiNE and its third-party dependencies retain their upstream licenses. See [NOTICE.md](NOTICE.md) before redistributing or modifying upstream components.
