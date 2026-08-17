# Deployment and Operations

## Host prerequisites

This is for RHEL 10 ARM64. Install Docker Engine and Compose:

```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker cloud-user
```

Use this persistent layout:

```text
/home/cloud-user/docker/redbot/{image,config,data,scripts,backups,logs}
/home/cloud-user/docker/reddash/image
```

`data` is critical Redbot state. It contains core configuration, installed cog code, Downloader libraries, and cog settings. Never commit it. Retain the `:Z` suffix on RHEL bind mounts. After restoring data run:

```bash
sudo chown -R 1000:1000 /home/cloud-user/docker/redbot/data
sudo restorecon -RFv /home/cloud-user/docker/redbot/data
```

## Redbot

Copy the files from `redbot/` into `/home/cloud-user/docker/redbot/`, put the Dockerfile in `image/`, and make the Chromium wrapper executable. Edit only `REPLACE_WITH_DISCORD_USER_ID` in the deployed Compose file. Keep `config/config.json` private and off Git.

```bash
cd /home/cloud-user/docker/redbot
docker compose config
docker compose build --pull
docker compose up -d
docker compose logs -f redbot
```

The `--rpc` command argument is required by Red-Dashboard. A running container does not prove all cogs loaded; inspect startup output:

```bash
docker logs --since 15m redbot 2>&1 | grep -Ei 'loaded packages|failed to load package|error|traceback'
```

## Red-Dashboard

Copy `reddash/` into `/home/cloud-user/docker/reddash/`, then start it after Redbot:

```bash
cd /home/cloud-user/docker/reddash
docker compose config
docker compose build --pull
docker compose up -d
docker compose logs -f reddash
curl -I http://127.0.0.1:4238
```

It shares Redbot's network namespace and listens only through Redbot's loopback binding at `127.0.0.1:4238`. For Cloudflare Tunnel, publish a hostname to `http://127.0.0.1:4238`; do not open TCP 4238 in OCI or UFW. Protect the hostname with Cloudflare Access.

## Cogs

Downloaded cog libraries persist under `/data/cogs/Downloader/lib` inside the container. Install a dependency persistently with:

```bash
docker exec redbot python -m pip install --target /data/cogs/Downloader/lib PACKAGE_NAME
```

For ARM64 ButtonPoll charts, prefer a Matplotlib Agg implementation instead of Plotly/Kaleido/Chromium. Before any cog change, create a timestamped backup, compile the changed Python, restart the bot, read logs, and test in a non-critical channel.

```bash
docker exec redbot python -m pip install --target /data/cogs/Downloader/lib matplotlib
docker exec redbot sh -lc 'PYTHONPATH=/data/cogs/Downloader/lib python -m py_compile /data/cogs/CogManager/cogs/buttonpoll/poll.py'
cd /home/cloud-user/docker/redbot && docker compose up -d --force-recreate redbot
```

## Backups, updates, and rollback

Before upgrades:

```bash
cd /home/cloud-user/docker/redbot
stamp=$(date +%Y%m%d-%H%M%S)
tar --zstd -cpf "backups/redbot-data-$stamp.tar.zst" data config
sha256sum "backups/redbot-data-$stamp.tar.zst" > "backups/redbot-data-$stamp.tar.zst.sha256"
tar -tf "backups/redbot-data-$stamp.tar.zst" >/dev/null
```

The Dockerfiles and Compose files pin Redbot to `3.5.24` and Red-Dashboard to `1.8.2`.

| Command | Rebuilds image | Updates the pinned application version | Preserves `data` |
| --- | --- | --- | --- |
| `docker compose restart` | No | No | Yes |
| `docker compose up -d --build` | Yes | No | Yes |
| `docker compose up -d --force-recreate` | No | No | Yes |

Therefore rebuilding the Dockerfile **does not update Redbot**. It recreates an image that contains the same pinned version. To update intentionally: review release notes and cog compatibility; back up; change `REDBOT_VERSION` and `image: local/redbot:<version>` together; run `docker compose build --pull --no-cache`; recreate; then verify Redbot, Red-Dashboard, and every important cog. Keep the older image until validation passes.

Avoid `docker compose down -v` during normal work. Do not commit tokens, passwords, Discord IDs, `config.json`, data, or backup archives.
