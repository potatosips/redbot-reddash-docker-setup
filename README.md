# Redbot, Red-Dashboard, and Cog Docker Setup

Production-oriented Docker Compose setup for an ARM64 Linux host. The stack is ordered deliberately:

1. **Redbot** — Discord bot, persistent data, custom cogs, Downloader libraries, and the RPC server.
2. **Red-Dashboard** — dashboard sharing Redbot's network namespace.
3. **Cogs** — installation, persistence, updates, and the ARM64 ButtonPoll chart-rendering fix.

This repository contains no tokens, passwords, Redbot data, Discord IDs, Cloudflare tokens, or downloaded cog source. Keep those only in local `.env` files or in the persistent host data directory.

## Contents

```text
.
├── redbot/
│   ├── Dockerfile
│   ├── compose.yml
│   └── scripts/chromium-redbot
├── reddash/
│   ├── Dockerfile
│   └── compose.yml
├── docs/
│   └── DEPLOYMENT.md
└── .gitignore
```

## Important: does rebuilding update Redbot?

**No—not when a version is pinned.** The Redbot Dockerfile installs exactly the version supplied by `REDBOT_VERSION`; the provided Compose file pins it to `3.5.24`. Running `docker compose up -d --build` recreates the image, but it installs the same pinned Redbot version.

To intentionally update, change the version in `redbot/compose.yml`, review the upstream release notes, back up `data/`, rebuild, and check logs. The same policy applies to Red-Dashboard.

Never use an unpinned `pip install Red-DiscordBot` in a production image if predictable updates matter.

## Quick start

Read the documents in order:

1. [Deployment, cogs, backups, and controlled updates](docs/DEPLOYMENT.md)

The persistent runtime layout used by this guide is:

```text
/home/cloud-user/docker/
├── redbot/
│   ├── compose.yml
│   ├── image/Dockerfile
│   ├── config/
│   ├── data/                 # critical persistent Redbot state
│   ├── scripts/
│   └── backups/
└── reddash/
    ├── compose.yml
    └── image/Dockerfile
```

`data/` is intentionally outside the container image. Recreating a container or rebuilding an image does not delete it.
