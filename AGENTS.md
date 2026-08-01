# AGENTS.md — arc-infra

Orchestration + Postgres schema for the Arc stack. No business code.

## What lives here

- `docker-compose.yml` — references build contexts `../arc-gateway`, `../arc-core`, `../arc-web-admin`. Each upstream repo has its own Dockerfile.
- `docker/db/init.sql` — Postgres bootstrap (auto-loaded on first container start via `/docker-entrypoint-initdb.d/`).
- `docker/db/migrate.sql` — manual ALTER patches (NOT auto-loaded; run by hand against running DB).

## What does NOT live here

- No Rust, no Python source code, no React.
- No Dockerfile for any service. The Dockerfiles live in each upstream repo.

## Editing rules

- **Schema changes**: edit `docker/db/init.sql`. Both `arc-gateway` and `arc-core` consume this schema; coordinate PRs.
- **Service additions/changes**: edit `docker-compose.yml`. The relative paths `../arc-<svc>` are load-bearing.
- **Do not commit** `docker-compose.override.yml` or other local-only files. Use a `.env` (gitignored) for local tweaks.

## Dependencies on sibling repos

Each `build.context` must point at a directory that contains a working Dockerfile:

| Service | Context | Dockerfile |
|---|---|---|
| gateway | `../arc-gateway` | `docker/Dockerfile` |
| pose-service | `../arc-core` | `docker/Dockerfile` |
| celery-worker | `../arc-core` | `docker/Dockerfile` (override command in compose) |
| web-admin | `../arc-web-admin` | `Dockerfile` |

## Volume ownership

- `postgres_data` (named volume) — owned by arc-infra.
- `redis_data` (named volume) — owned by arc-infra.
- `data/videos` (host bind mount) — owned by arc-gateway (write) / arc-core (read+write). Mounted from `../arc-gateway/data/videos`.
- `models/` (host bind mount) — owned by arc-core. Mounted from `../arc-core/models`.

## Repo state on disk

This directory was created by splitting `/Users/garry/Projects/Arc/arc/docker/db/` and `docker-compose.yml` out of the original monorepo on 2026-08-01.