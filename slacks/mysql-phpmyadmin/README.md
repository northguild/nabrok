# MySQL + phpMyAdmin

A ready-to-run Docker Compose stack for MySQL with the phpMyAdmin web interface.

## Setup

Grab these two files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/mysql-phpmyadmin/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/mysql-phpmyadmin/.env.example) | Environment variables (copy to `.env`) |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/mysql-phpmyadmin/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/mysql-phpmyadmin/.env.example

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

- **phpMyAdmin UI**: http://localhost:8080
  - Username: `root`
  - Password: from `MYSQL_ROOT_PASSWORD` (default: `root`)
- **MySQL**: `localhost:3306`
  - Root: username `root`, password from `MYSQL_ROOT_PASSWORD`
  - App: username from `MYSQL_USER`, password from `MYSQL_PASSWORD`

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
