# MySQL + phpMyAdmin

A ready-to-run Docker Compose stack for MySQL with the phpMyAdmin web interface.

## Quick Start

```bash
cd mysql-phpmyadmin

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
