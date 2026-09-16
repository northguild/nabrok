# Prometheus + Grafana Cheatsheet

Quick reference for Prometheus metrics, PromQL queries, and Grafana dashboards.

## What is Prometheus?

Prometheus is an open-source **metrics collection and monitoring system**. It:

1. **Scrapes** metrics from services (HTTP endpoints that return numbers)
2. **Stores** them as time-series data (metric name + labels + timestamp + value)
3. **Queries** them with PromQL (Prometheus Query Language)
4. **Alerts** when thresholds are crossed

## Prometheus Architecture

```
┌─────────────┐     scrape      ┌──────────────┐
│  Target     │ ───────────────→│  Prometheus  │
│  (your app) │   HTTP /metrics │  Server      │
└─────────────┘                 └──────┬───────┘
                                       │ stores
                                       ▼
                                ┌──────────────┐     query    ┌──────────┐
                                │  Time-Series  │ ←───────── │ Grafana  │
                                │   Database    │   PromQL   │ Dashboard│
                                └──────────────┘             └──────────┘
```

## Understanding Metrics

Prometheus metrics have 4 types:

| Type | Description | Example |
|------|-------------|---------|
| **Counter** | Only goes up (or resets) | `http_requests_total` |
| **Gauge** | Goes up and down | `cpu_temperature_celsius` |
| **Histogram** | Distribution of values | `http_request_duration_seconds` |
| **Summary** | Similar to histogram | `api_response_time` |

## PromQL Basics

### Basic Queries

```promql
# Get all metrics
up

# Get a specific metric
http_requests_total

# Filter by label
http_requests_total{method="GET"}

# Multiple label filters
http_requests_total{method="GET", handler="/api"}

# Time range selector (last 5 minutes)
http_requests_total[5m]
```

### Aggregation Functions

```promql
# Sum across all labels
sum(http_requests_total)

# Sum by specific label (group by handler)
sum by (handler) (http_requests_total)

# Count instances
count(up)

# Average value
avg(node_memory_MemAvailable_bytes)

# Top 10 by metric
topk(10, http_requests_total)

# Rate of change per second (last 5 minutes)
rate(http_requests_total[5m])

# Increase over time period
increase(http_requests_total[1h])
```

### Common Patterns

```promql
# Requests per second by handler
sum by (handler) (rate(http_requests_total[5m]))

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m])) * 100

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# CPU usage (if you have node_cpu_seconds_total)
1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))

# Active connections
sum(up)

# Pending requests
queue_length
```

### Vector Math

```promql
# Subtract two metrics
http_requests_total{method="GET"} - http_requests_total{method="POST"}

# Multiply by constant
rate(http_requests_total[5m]) * 1024  # convert to KB/s
```

### Functions

```promql
# Rate of change (per second)
rate(metric[5m])

# Increase over period
increase(metric[1h])

# Average over time window
avg_over_time(metric[1h])

# Percentile (95th percentile)
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Time shift (compare with 1 hour ago)
rate(http_requests_total[5m]) offset 1h
```

## Prometheus UI (http://localhost:9090)

- **Graph**: Visualize query results over time
- **Console**: See raw metric values right now
- **Status → Targets**: See which services are being scraped
- **Status → Config**: View current configuration
- **Alerts**: See defined alert rules

## Grafana Basics

### Adding Prometheus as a Data Source

1. Go to **Connections → Data sources**
2. Click **Add data source** → Select **Prometheus**
3. Set URL: `http://prometheus:9090` (Docker internal) or `http://localhost:9090` (browser)
4. Click **Save & test**

### Creating a Dashboard

1. Go to **Dashboards → New dashboard**
2. Click **Add visualization**
3. Write a PromQL query
4. Choose visualization type (graph, stat, table, etc.)
5. Set title and save

### Popular Pre-built Dashboards

| Dashboard ID | Description |
|-------------|-------------|
| `3662` | Prometheus Stats |
| `1860` | Node Exporter Full (system metrics) |
| `405` | Go Runtime Metrics |
| `15760` | Redis Overview |

Import: **Dashboards → Import → Enter ID**

### Panel Tips

- **Time range**: Use the top-right date picker to change time window
- **Refresh**: Click the refresh icon to reload data
- **Explore**: Go to **Explore** for ad-hoc queries with live results
- **Annotations**: Add markers for deployments/events on graphs

## Quick Start: Monitor Your Own App

### Step 1: Expose `/metrics` endpoint

Your app should expose an HTTP endpoint that returns metrics in Prometheus format:

```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",handler="/api"} 1234
http_requests_total{method="POST",handler="/api"} 567
```

### Step 2: Add scrape config to `prometheus.yml`

```yaml
scrape_configs:
  - job_name: "my-app"
    static_configs:
      - targets: ["host.docker.internal:8080"]
```

### Step 3: Query in Grafana

```promql
# Requests per second
sum by (handler) (rate(http_requests_total[5m]))

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))
```

## Useful Dashboard Ideas

1. **Request Rate**: `sum by (handler) (rate(http_requests_total[5m]))`
2. **Error Rate**: `sum(rate(http_requests_total{status=~"5.."}[5m])) * 100`
3. **P95 Latency**: `histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))`
4. **Active Users**: `count(unique(user_id))`
5. **Queue Depth**: `queue_length`

## Stop & Clean Up

```bash
docker compose down          # stop containers
docker compose down -v       # stop and remove volumes (wipes data)
```
