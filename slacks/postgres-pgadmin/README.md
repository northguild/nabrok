# PostgreSQL + pgAdmin

A ready-to-run Docker Compose stack for PostgreSQL with the official pgAdmin 4 web interface.

## Quick Start

```bash
cd postgres-pgadmin

# Optional: customize credentials
cp .env.example .env
# Edit .env to your liking

docker compose up -d
```

## Access

- **pgAdmin UI**: http://localhost:5050
  - Email: `admin@nabrok.dev` (or set via `PGADMIN_EMAIL`)
  - Password: `admin` (or set via `PGADMIN_PASSWORD`)
- **PostgreSQL**: `localhost:5432`
  - User: `postgres` (or set via `POSTGRES_USER`)
  - Password: `postgres` (or set via `POSTGRES_PASSWORD`)

## Add a Server in pgAdmin

After logging in, right-click **Servers → Create → Server** and fill in:

| Field       | Value                  |
|-------------|------------------------|
| Host        | `postgres`             |
| Port        | `5432`                 |
| Username    | from `POSTGRES_USER`   |
| Password    | from `POSTGRES_PASSWORD` |

> The service name `postgres` is used as the hostname inside Docker's network.

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
