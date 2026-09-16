# MinIO + Console

A ready-to-run Docker Compose stack for MinIO — an S3-compatible object storage server.

## Setup

Grab these two files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/minio-console/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/minio-console/.env.example) | Environment variables (copy to `.env`) |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/minio-console/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/minio-console/.env.example

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

- **MinIO Console (Web UI)**: http://localhost:9001
  - Username: from `MINIO_ROOT_USER` (default: `minioadmin`)
  - Password: from `MINIO_ROOT_PASSWORD` (default: `minioadmin`)
- **MinIO API (S3 endpoint)**: http://localhost:9000
  - Region: `us-east-1` (default)

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
