# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal home-lab infrastructure as code. Single `docker-compose.infra.yml` deployed as a Docker Swarm stack named `infra`.

## Deployment

No local build step. Deploy via GitHub Actions:

- **Manual deploy**: trigger `Deploy Rulyotano (infra)` workflow in GitHub Actions (`workflow_dispatch`).
- Workflow SSHs into the manager node, downloads the compose file directly from `main` branch on GitHub, then runs `docker stack deploy -c docker-compose.yml infra`.
- **Changes must be pushed to `main`** before triggering deployment — the workflow fetches the raw file from GitHub, not from local.

To back up a Docker volume, trigger `Volume Backup` workflow with `serverName`, `volumeName`, and `volumePath`. Backup is uploaded as a GitHub Actions artifact (1-day retention).

## Architecture

Two-node Docker Swarm:

| Node | Role | Services |
|------|------|----------|
| `rulyotano1` | Swarm manager | Traefik (reverse proxy), Redis |
| `rulyotano2` | Worker | PostgreSQL |

**Networks** (both external, must exist on swarm before deploy):
- `traefik` — public-facing, Traefik routes here
- `backend` — internal service communication

**Volumes** (all external):
- `postgres_data`, `redis_storage`

**Secrets** (Docker Swarm secrets, external):
- `postgres_user`, `postgres_password`

**GitHub Actions secrets required**:
- `REMOTE_HOST` — manager node address
- `REMOTE_USER` — SSH user
- `DOCKER_SSH_PRIVATE_KEY` — SSH private key

## Traefik

- HTTP → 80, HTTPS → 443, FTP → 21
- ACME via HTTP challenge, email `contact@rulyotano.com`, stored in `acme.json` (inside container)
- Dashboard at `traefik.minesweeper.rulyotano.com`, protected by basic auth
- Services opt-in via Docker labels; `traefik.enable=false` on internal services (Redis, Postgres)
- Pinned to manager node (`node.role==manager && node.hostname==rulyotano1`)

## Adding a new service

1. Add service to `docker-compose.infra.yml` under `services:`.
2. Attach to `traefik` network if it needs external routing, `backend` if it only talks to other services.
3. Add Traefik labels for routing/TLS if public-facing.
4. If it needs a persistent volume or secret, declare them as `external: true` and create them on the swarm first.
5. Push to `main`, trigger deploy workflow.
