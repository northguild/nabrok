# MongoDB + MongoDB Express

A ready-to-run Docker Compose stack for MongoDB with the MongoDB Express web interface.

## Quick Start

```bash
cd mongodb-express

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
