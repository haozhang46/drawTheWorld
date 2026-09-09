# 12 — K8s observability (metrics + logs)

Status: ready-for-human  
Type: task  
Blocked by: 02

## Goal

Production observability on Kubernetes with **nginx Ingress as the edge**: metrics for performance, Loki for logs only. No full APM/traces in this ticket.

## Scope

1. **nginx Ingress Controller** — cluster HTTP(S) entry; TLS terminate at Ingress; route to `dtw-api` Service. Enable prometheus metrics + access logs.
2. **kube-prometheus-stack** — Prometheus + Grafana + Alertmanager, plus node / Pod metrics (CPU, memory, disk, restarts). Scrape nginx Ingress metrics (request rate, latency, upstream 5xx).
3. **Hono `/metrics`** — Prometheus scrape endpoint: request latency (p50/p95/p99), QPS by route, error counts (HTTP 5xx + domain `error.code` where useful).
4. **Postgres hosted monitoring** — use the managed provider’s dashboards/alerts for connections, disk, CPU (no in-cluster Postgres as the HA source of truth).
5. **Loki** — logs only (API business JSON on stdout + **nginx access/error logs**); correlate edge ↔ app via `requestId` / `X-Request-Id` in Grafana, not a substitute for performance signals.

## Edge notes

- Propagate / generate `X-Request-Id` at nginx so access log and Hono logs share one id.
- Do not expose `/metrics` publicly via Ingress (ClusterIP scrape or network policy only).
- Health: Ingress backends use API readiness (`/health/ready`); keep `/metrics` off the public host if possible.

## Out of scope

- OpenTelemetry / Tempo / Jaeger traces (later if Save/Nearby latency needs deep drill-down)
- In-cluster HA Postgres
- Replacing managed object-storage monitoring
- Alternate gateways (Envoy / Traefik) unless nginx is unavailable on the target cluster

## Acceptance

- [ ] nginx Ingress Controller installed; TLS Ingress routes to API Service
- [ ] nginx Ingress metrics scraped; Grafana can show edge QPS / latency / upstream errors
- [ ] kube-prometheus-stack (or equivalent) installed; Grafana can show node/Pod CPU, memory, disk
- [ ] API exposes scrapeable `/metrics`; ServiceMonitor or scrape config wired; **not** public via Ingress
- [ ] Alertmanager (or provider alerts) covers: Ingress/API 5xx / high latency, Pod not Ready, node/PVC disk high
- [ ] Managed Postgres alerts for connection saturation, disk, CPU documented or enabled
- [ ] Fluent Bit (or equivalent) → Loki for API stdout + nginx access logs; `X-Request-Id` / `requestId` documented
- [ ] Short note under `docs/` or runbook: nginx edge + metrics vs Loki roles + where dashboards live

## Comments

- 2026-09-09: Locked stack from ops discussion — Prometheus for performance, Loki for logs, managed Postgres for DB signals.
- 2026-09-09: Entry = nginx Ingress Controller; scrape its metrics; ship access logs to Loki; correlate with Hono via request id.
