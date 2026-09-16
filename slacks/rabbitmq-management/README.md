# RabbitMQ + Management

A ready-to-run Docker Compose stack for RabbitMQ — a robust message broker — with the built-in management web UI.

## Setup

Grab these two files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/rabbitmq-management/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/rabbitmq-management/.env.example) | Environment variables (copy to `.env`) |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/rabbitmq-management/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/rabbitmq-management/.env.example

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

- **Management UI**: http://localhost:15672
  - Username: from `RABBITMQ_USER` (default: `guest`)
  - Password: from `RABBITMQ_PASS` (default: `guest`)
- **AMQP Protocol**: `localhost:5672`

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
