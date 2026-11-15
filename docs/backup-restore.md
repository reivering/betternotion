# Back up and restore

Stop the app first so writes finish, then export the database and archive the files and configuration. Run these commands from the repository directory:

```sh
mkdir -p backups
chmod 700 backups
umask 077
docker compose stop affine
docker compose exec -T postgres pg_dump -U affine -d affine --format=custom > backups/postgres.dump
tar -czf backups/files-and-config.tar.gz config data/storage .env
chmod 600 backups/postgres.dump backups/files-and-config.tar.gz
docker compose start affine
```

The backups contain private notes, uploaded files, server settings, and the database password. Store them encrypted somewhere separate from the server. Keep the same `.env` password for restore.

To restore to a fresh deployment, stop the app and database, extract the files-and-config archive in the repository directory, then initialize PostgreSQL and restore the dump:

```sh
docker compose down
tar -xzf backups/files-and-config.tar.gz -C .
docker compose up -d postgres
until docker compose exec -T postgres pg_isready -U affine -d affine; do sleep 2; done
docker compose exec -T postgres pg_restore -U affine -d affine --clean --if-exists --no-owner < backups/postgres.dump
docker compose up -d
```

Open the app and check several pages and attachments before deleting the backup. Test this procedure on a separate deployment before relying on it for a production recovery.
