# Prometheus + Grafana

A ready-to-run Docker Compose stack for monitoring with Prometheus (metrics collection) and Grafana (visualization dashboards).

## Setup

Grab these files into your own project — no need to clone the whole repo:

| File | Purpose |
|------|---------|
| [`docker-compose.yml`](slacks/prometheus-grafana/docker-compose.yml) | The stack definition |
| [`.env.example`](slacks/prometheus-grafana/.env.example) | Environment variables (copy to `.env`) |
| [`prometheus.yml`](slacks/prometheus-grafana/prometheus.yml) | Prometheus configuration |
| [`provisioning/`](slacks/prometheus-grafana/provisioning/) | Grafana auto-provisioning config |

```bash
# Copy the files into your project
curl -o docker-compose.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/prometheus-grafana/docker-compose.yml
curl -o .env.example https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/prometheus-grafana/.env.example
curl -o prometheus.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/prometheus-grafana/prometheus.yml
mkdir -p provisioning/datasources
curl -o provisioning/datasources/prometheus.yml https://raw.githubusercontent.com/<your-org>/nabrok/main/slacks/prometheus-grafana/provisioning/datasources/prometheus.yml

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

- **Grafana UI**: http://localhost:3000
  - Username: from `GRAFANA_USER` (default: `admin`)
  - Password: from `GRAFANA_PASS` (default: `admin`)
- **Prometheus UI**: http://localhost:9090

## First Steps in Grafana

1. Log in to Grafana at http://localhost:3000
2. Go to **Connections → Data sources → Add data source**
3. Select **Prometheus**
4. Set URL to `http://prometheus:9090` (use the Docker service name)
5. Click **Save & test**
6. Import a dashboard: **Dashboards → Import → Upload** or use ID `3662` (Prometheus stats)

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
