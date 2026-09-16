# MongoDB + MongoDB Express

A ready-to-run Docker Compose stack for MongoDB with the MongoDB Express web interface.

## Setup

Grab these two files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/mongodb-express/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/mongodb-express/.env.example) | Environment variables (copy to `.env`) |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/mongodb-express/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/mongodb-express/.env.example

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

- **MongoDB Express UI**: http://localhost:8081
  - Username: from `ME_USER` (default: `admin`)
  - Password: from `ME_PASSWORD` (default: `admin`)
- **MongoDB**: `localhost:27017`
  - Root username: from `MONGO_ROOT_USER` (default: `admin`)
  - Root password: from `MONGO_ROOT_PASSWORD` (default: `admin`)

## Connection String

```
mongodb://admin:admin@localhost:27017/
```

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
