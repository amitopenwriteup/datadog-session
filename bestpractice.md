# Datadog Best Practices Workshop

A practical guide to observability, monitoring, and alerting with Datadog.

---

## Table of Contents

1. [Agent & Installation](#1-agent--installation)
2. [Tagging Strategy](#2-tagging-strategy)
3. [Metrics & Custom Metrics](#3-metrics--custom-metrics)
4. [Logs Management](#4-logs-management)
5. [APM & Distributed Tracing](#5-apm--distributed-tracing)
6. [Dashboards](#6-dashboards)
7. [Monitors & Alerting](#7-monitors--alerting)
8. [Infrastructure Monitoring](#8-infrastructure-monitoring)
9. [Synthetics & RUM](#9-synthetics--rum)
10. [Security & Access Control](#10-security--access-control)
11. [Cost Optimization](#11-cost-optimization)

---

## 1. Agent & Installation

### Keep the Agent Up to Date
- Always run the latest stable Datadog Agent (v7+).
- Use a configuration management tool (Ansible, Puppet, Chef, Terraform) to deploy and update agents consistently.
- Avoid installing the agent manually on individual hosts in production.

### Agent Configuration
- Store all agent config in `/etc/datadog-agent/datadog.yaml` (Linux) or equivalent.
- Never hardcode API keys in code. Use environment variables or secrets managers (Vault, AWS Secrets Manager).
- Enable only the integrations you need — unnecessary checks add overhead.

### Containerized Environments
- Use the official `datadog/agent` Docker image or the Datadog Helm chart for Kubernetes.
- Deploy as a DaemonSet in Kubernetes to ensure every node is covered.
- Enable the Cluster Agent for Kubernetes metrics aggregation and autoscaling.
- Use Autodiscovery annotations to dynamically configure integrations.

```yaml
# Example Kubernetes pod annotation for Autodiscovery
annotations:
  ad.datadoghq.com/redis.check_names: '["redisdb"]'
  ad.datadoghq.com/redis.init_configs: '[{}]'
  ad.datadoghq.com/redis.instances: '[{"host": "%%host%%", "port": "6379"}]'
```

---

## 2. Tagging Strategy

Tags are the foundation of everything in Datadog. A poor tagging strategy leads to ungroupable metrics, useless dashboards, and broken alerts.

### Unified Service Tagging (UST)
Apply these three tags **everywhere** — hosts, containers, logs, traces, and metrics:

| Tag | Example | Purpose |
|-----|---------|---------|
| `env` | `env:production` | Separate environments |
| `service` | `service:checkout-api` | Identify the service |
| `version` | `version:1.4.2` | Track deployments |

### Additional Recommended Tags

```
team:platform
region:us-east-1
datacenter:aws-virginia
app:storefront
tier:backend
```

### Rules
- Use lowercase, snake_case or kebab-case consistently — pick one and enforce it.
- Avoid high-cardinality tag values (e.g., user IDs, request IDs, timestamps).
- Define your tagging taxonomy in a shared doc before rollout. Tag sprawl is hard to fix retroactively.
- Apply tags at the infrastructure level (cloud provider, configuration management) rather than relying on individual teams.

---

## 3. Metrics & Custom Metrics

### Types of Metrics
- **Gauge** — a value at a point in time (e.g., CPU usage, queue depth).
- **Count** — a number of events in an interval (e.g., requests per second).
- **Rate** — count normalized per second.
- **Histogram** — distribution of values (e.g., response time percentiles).
- **Distribution** — globally accurate percentiles across all agents.

### Custom Metrics Best Practices
- Prefer **Distribution** metrics over Histograms for latency — they support accurate p95/p99 globally.
- Avoid tag combinations that explode cardinality. Each unique tag combination = one custom metric.
- Use `DogStatsD` for high-throughput custom instrumentation.
- Audit your custom metric count regularly in **Metrics > Summary**.

```python
# DogStatsD example — Python
from datadog import statsd

statsd.increment('web.page_views', tags=["page:home", "env:production"])
statsd.histogram('web.response_time', 0.123, tags=["endpoint:/checkout"])
```

### Metric Naming Convention
```
<namespace>.<entity>.<measurement>
# Examples:
app.orders.processed
api.latency.p95
db.connections.active
```

---

## 4. Logs Management

### Structured Logging
- Always emit logs in **JSON format**. Structured logs are parseable without custom pipelines.
- Include standard fields in every log line:

```json
{
  "timestamp": "2024-11-15T10:32:00Z",
  "level": "error",
  "service": "checkout-api",
  "env": "production",
  "version": "1.4.2",
  "message": "Payment gateway timeout",
  "trace_id": "abc123",
  "span_id": "def456",
  "user_id": "u_89012"
}
```

### Log Pipelines
- Use **Pipelines** to parse, remap, and enrich logs centrally — avoid parsing in every service.
- Remap `timestamp`, `service`, `status`, `trace_id`, and `span_id` to Datadog's reserved attributes.
- Use **Grok Parser** for legacy non-JSON logs.

### Indexes & Retention
- Separate logs by `env` and `service` into different indexes with different retention periods.
- Use **Exclusion Filters** to drop noisy, low-value logs (e.g., health check hits) before indexing.
- Route high-volume debug logs to **Flex Logs** or **Archives** (S3/GCS) for cost efficiency.

### Log-Based Metrics
- Generate metrics from logs for long-term trend analysis (logs are expensive to retain long-term).
- Example: count of `status:error` logs per `service` per minute → stored as a metric indefinitely.

---

## 5. APM & Distributed Tracing

### Instrumentation
- Use **automatic instrumentation** via the Datadog tracing library for your language — it covers most frameworks with zero code changes.
- Add **manual spans** for critical business logic not covered by auto-instrumentation.

```python
# Manual span — Python
from ddtrace import tracer

with tracer.trace("checkout.validate_cart", service="checkout-api") as span:
    span.set_tag("cart.item_count", len(cart.items))
    result = validate_cart(cart)
```

### Connect Logs, Traces, and Metrics
- Enable **Log and Trace correlation** by injecting `trace_id` and `span_id` into logs.
- Use Unified Service Tagging so you can jump from a metric spike → trace → log seamlessly.

### Service Map
- Review the **Service Map** regularly to discover undocumented dependencies.
- Use `peer.service` tags to label external dependencies (third-party APIs, managed DBs).

### Sampling
- Use **Head-based sampling** for high-traffic services to control ingestion costs.
- Use **Error & rare sampling** rules to ensure 100% of error traces are retained regardless of sampling.
- Set sampling rules per service/environment, not globally.

```yaml
# datadog.yaml — sampling rules
apm_config:
  trace_sampling_rules:
    - service: "checkout-api"
      sample_rate: 0.1  # 10% of traces
    - service: "checkout-api"
      name: "checkout.process_payment"
      sample_rate: 1.0  # 100% of payment spans
```

---

## 6. Dashboards

### Design Principles
- Every dashboard should answer a specific question. Avoid generic "all metrics" dashboards.
- Group widgets by functional area with visual section headers.
- Pin your most critical dashboards to the navigation.

### Dashboard Types

| Type | Use Case |
|------|----------|
| **Screenboard** | Live NOC views, status boards |
| **Timeboard** | Correlated time-series investigation |
| **Service Summary** | Per-service health (auto-generated by APM) |

### Widget Best Practices
- Use **template variables** (`$env`, `$service`) so one dashboard works across all environments.
- Always set **meaningful titles** on widgets — the metric name alone is not a title.
- Use **Change** and **Top List** widgets to surface anomalies, not just current values.
- Add **Event Overlays** to timeseries graphs to correlate spikes with deployments or config changes.
- Prefer **p95/p99** for latency over averages — averages hide tail latency.

### Recommended Dashboards to Build
1. **Service Health** — RED metrics (Rate, Errors, Duration) per service
2. **Infrastructure Overview** — CPU, memory, disk, network by host/pod
3. **Business KPIs** — orders, signups, revenue (from logs/custom metrics)
4. **On-Call Overview** — active alerts, recent events, SLO status
5. **Deployment Tracker** — version rollout, error rate change, latency change

---

## 7. Monitors & Alerting

### Alert Philosophy
- Every alert must be **actionable**. If no one knows what to do, delete the alert.
- Prefer **fewer, higher-quality alerts** over alert sprawl.
- Use **priority levels**: P1 (wake someone up) vs P3 (review in the morning).

### Monitor Types

| Type | When to Use |
|------|------------|
| **Metric** | Threshold-based, e.g., CPU > 90% |
| **Anomaly** | When normal baseline fluctuates (e.g., traffic) |
| **Forecast** | Predict when a resource will be exhausted |
| **Composite** | Combine conditions to reduce false positives |
| **SLO Alert** | Alert when error budget is burning too fast |
| **Log** | Alert on log patterns or error counts |

### Configuration Best Practices

```
Evaluation window: 5 minutes (avoid 1m — too noisy)
Renotify: Only if still alerting after 30+ minutes
Recovery threshold: Set lower than alert threshold to prevent flapping
Message: Always include a Runbook link
```

### Notification Message Template

```
## {{monitor.name}} — {{#is_alert}}ALERT{{/is_alert}}{{#is_warning}}WARNING{{/is_warning}}

**Service:** {{service.name}}
**Environment:** {{env.name}}
**Value:** {{value}}

### What happened
Describe the symptom in plain English.

### Impact
Who or what is affected?

### Steps to investigate
1. [Check APM traces](https://app.datadoghq.com/apm/services/{{service.name}})
2. [View logs](https://app.datadoghq.com/logs?query=service:{{service.name}})
3. [Runbook](https://wiki.internal/runbooks/{{service.name}})

@pagerduty-oncall
```

### Downtime & Maintenance
- Always schedule **Downtime** windows before planned maintenance — never silence alerts manually ad hoc.
- Use **Monitor Tags** to bulk-silence all monitors for a service during a deployment.

---

## 8. Infrastructure Monitoring

### Host Monitoring
- Set alerts on `system.cpu.user > 85%`, `system.mem.pct_usable < 10%`, and `system.disk.in_use > 85%`.
- Use **process monitors** to alert if a critical process is not running.
- Enable the **Network Performance Monitoring** (NPM) integration for east-west traffic visibility.

### Kubernetes
- Monitor the **Kubernetes State Metrics** integration — covers pod restarts, pending pods, and resource limits.
- Alert on `kubernetes.pods.running` drops and `kubernetes.containers.restarts` spikes.
- Use the **Live Containers** view for real-time debugging.
- Set resource requests and limits on all pods, and monitor actual vs requested usage.

### Cloud Integrations
- Enable native cloud integrations (AWS, GCP, Azure) via the Datadog console — these pull in CloudWatch/Stackdriver metrics automatically.
- For AWS: enable the **Enhanced CloudWatch metrics** for RDS, ElastiCache, and ECS.
- Tag cloud resources consistently so Datadog can correlate them with your application metrics.

---

## 9. Synthetics & RUM

### Synthetic Monitoring
- Create **API tests** for every critical external endpoint. Run from multiple locations.
- Create **Browser tests** for critical user journeys (login, checkout, signup).
- Set test frequency to 1 minute for P1 flows, 5 minutes for others.
- Use **Multistep API tests** for authenticated flows that require session state.

```yaml
# Example: API test assertion
assertions:
  - type: statusCode
    operator: is
    target: 200
  - type: responseTime
    operator: lessThan
    target: 2000  # ms
  - type: body
    operator: contains
    target: '"status":"ok"'
```

### Real User Monitoring (RUM)
- Install the RUM SDK on all customer-facing frontends.
- Enable **Session Replay** for debugging user-reported issues (ensure PII masking is configured).
- Track **Core Web Vitals** (LCP, FID, CLS) as SLIs.
- Create RUM-based monitors on `Largest Contentful Paint > 2.5s` and JS error rate spikes.

---

## 10. Security & Access Control

### API Keys & App Keys
- Use separate API keys per environment (prod/staging/dev). Never share keys across environments.
- Rotate keys every 90 days or immediately after any suspected exposure.
- Store keys in a secrets manager — never in version control or environment files committed to git.

### Role-Based Access Control (RBAC)
- Create custom roles with least-privilege access. Avoid assigning the built-in Admin role broadly.
- Use **Teams** to manage dashboard, monitor, and notebook ownership.
- Enable **Audit Trail** to track who accessed or changed what.

### Sensitive Data Scanner
- Enable the **Sensitive Data Scanner** on log pipelines to detect and redact PII, credentials, and tokens before they are indexed.
- Define custom scanning rules for your internal data formats (e.g., internal user IDs, tokens).

---

## 11. Cost Optimization

### Understand Your Bill
- Review **Plan & Usage** monthly. Costs come from: hosts, custom metrics, APM hosts, log ingested/indexed, synthetics, and RUM sessions.
- The biggest cost levers are usually **custom metrics** and **log indexing**.

### Reduce Custom Metrics
- Audit metrics with the **Metrics Without Limits™** feature — collect all metrics but only index what you query.
- Remove unused metrics from dashboards and monitors, then exclude them from indexing.
- Consolidate high-cardinality tags.

### Reduce Log Costs
- Drop health check and noise logs at the pipeline level with Exclusion Filters.
- Use **Flex Logs** for compliance/audit logs that are rarely queried.
- Archive to S3/GCS and use **Log Rehydration** for ad-hoc historical queries.

### APM
- Tune sampling rates — most services don't need 100% trace ingestion.
- Use **Ingestion Controls** to cap ingestion per service.

---

## Quick Reference Checklist

### New Service Onboarding
- [ ] Unified Service Tagging applied (`env`, `service`, `version`)
- [ ] Datadog Agent / tracing library installed
- [ ] Structured JSON logging enabled with `trace_id` injection
- [ ] APM auto-instrumentation enabled
- [ ] RED metrics dashboard created
- [ ] Monitors created for error rate, latency p95, and throughput
- [ ] Synthetic API test created for health endpoint
- [ ] On-call runbook linked in monitor messages
- [ ] SLO defined and configured

### Monthly Hygiene
- [ ] Review custom metric count and remove unused metrics
- [ ] Check log index volumes and update exclusion filters
- [ ] Audit stale monitors (no alerts in 90 days)
- [ ] Review and rotate API keys
- [ ] Check SLO compliance and error budget burn
- [ ] Update dashboard template variables for any new services/environments

---

*Last updated: June 2026 | Based on Datadog Agent v7 and current platform features.*
