# Monitoring and Logging

This homelab uses a centralized monitoring and logging stack to provide visibility into infrastructure health, service availability, performance metrics, and application logs. Alerts are delivered through Telegram for rapid incident awareness.

## Architecture

```text
┌──────────────────────┐
│   Services &         │
│   Infrastructure     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Grafana Alloy      │
│   Metrics & Logs     │
│   Collection         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│        Loki          │
│        Logs          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       Grafana        │
│   Dashboards         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│     Uptime Kuma      │
│   Availability       │
│   Monitoring         │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Telegram Alerts    │
│   Notifications      │
└──────────────────────┘
```

## Stack Summary

| Component     | Purpose                                       |
| ------------- | --------------------------------------------- |
| Uptime Kuma   | Service uptime and availability monitoring    |
| Grafana Alloy | Collection and forwarding of metrics and logs |
| Loki          | Centralized log aggregation and storage       |
| Grafana       | Dashboards, visualization, and observability  |
| Telegram      | Alert notifications and incident awareness    |

## Key Capabilities

* Infrastructure and service monitoring
* Centralized log aggregation
* Uptime and availability monitoring
* Real-time alerting through Telegram
* Historical performance and troubleshooting data

