# Redis + Redis Commander

A ready-to-run Docker Compose stack for Redis with the Redis Commander web interface.

## Setup

Grab these two files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/redis-commander/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/redis-commander/.env.example) | Environment variables (copy to `.env`) |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/redis-commander/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/redis-commander/.env.example

# Or just download them from GitHub and move them wherever you like
```

## Quick Start

```bash
# Optional: customize credentials
cp .env.example .env
# Edit .env to your liking

docker compose up -d
```

## Access

- **Redis Commander UI**: http://localhost:5000
  - Username: from `REDIS_CMD_USER` (default: `admin`)
  - Password: from `REDIS_CMD_PASS` (default: `admin`)
- **Redis**: `localhost:6379`

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
