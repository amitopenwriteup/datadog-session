# 🐕 Datadog Monitoring Workshop
### A Hands-On Guide to Observability & Alerting

---

## Table of Contents

1. [Introduction to Datadog Monitoring](#1-introduction-to-datadog-monitoring)
2. [Understanding the Monitoring Dashboard](#2-understanding-the-monitoring-dashboard)
3. [Creating Your First Monitor](#3-creating-your-first-monitor)
4. [Monitor Types Deep Dive](#4-monitor-types-deep-dive)
5. [Triggered Monitors & Alerting](#5-triggered-monitors--alerting)
6. [Downtimes & Maintenance Windows](#6-downtimes--maintenance-windows)
7. [SLOs — Service Level Objectives](#7-slos--service-level-objectives)
8. [Event Management](#8-event-management)
9. [Monitor Quality & Best Practices](#9-monitor-quality--best-practices)
10. [Notification Setup](#10-notification-setup)
11. [Monitor Packs & Templates](#11-monitor-packs--templates)
12. [Hands-On Labs](#12-hands-on-labs)

---

## 1. Introduction to Datadog Monitoring

Datadog is a cloud-based monitoring and analytics platform that provides full-stack observability. The **Monitoring** module is the core component for setting up alerts, tracking SLOs, and managing incidents.

### Key Concepts

| Term | Description |
|------|-------------|
| **Monitor** | A rule that watches a metric, log, or trace and alerts when a threshold is breached |
| **Alert** | A notification sent when a monitor enters a triggered state |
| **Downtime** | A scheduled silence period to mute monitor alerts |
| **SLO** | Service Level Objective — a measurable reliability target |
| **Watchdog** | AI-powered anomaly detection that auto-discovers issues |

### Monitor States

```
OK  →  WARN  →  ALERT  →  NO DATA
 ↑___________________________________↓ (recovery)
```

- **OK** — Metric is within acceptable range
- **WARN** — Approaching threshold (optional)
- **ALERT** — Threshold breached — notifications fire
- **NO DATA** — Agent or metric source stopped reporting

---

## 2. Understanding the Monitoring Dashboard

### Navigation Overview

When you open **Monitoring** in Datadog, the left sidebar provides:

```
Monitoring
├── Monitor List       ← View all monitors
├── New Monitor        ← Create a monitor
│   └── Triggered      ← Currently firing monitors
│   └── Downtimes      ← Scheduled silences
│   └── Quality        ← Monitor health score
├── SLOs               ← Service Level Objectives
├── Check Summary      ← Agent check statuses
└── Event Management
    ├── Triage Inbox
    └── All Events
```

### Monitor List Columns

| Column | Meaning |
|--------|---------|
| **Status** | Current state: OK / ALERT / WARN / NO DATA / MUTED |
| **Priority** | P1 (critical) → P5 (low) |
| **Muted Left** | Time remaining on a downtime/mute |
| **Name** | Monitor name (e.g., `[Jenkins] Elevated Error Rate`) |
| **Tags** | Labels like `service:jenkins`, `env:prod` |

### Reading the Status Bar

The bar chart at the top of Monitor List shows monitor state distribution over time. Spikes indicate incident periods. Use it to spot recurring alert patterns.

---

## 3. Creating Your First Monitor

### Step-by-Step: Metric Monitor

**Navigate to:** Monitoring → New Monitor → Metric

#### Step 1 — Choose Detection Method

```
Threshold    ← Alert when value crosses a fixed number
Change       ← Alert on rate of change
Anomaly      ← Alert on unusual patterns (ML-based)
Outlier      ← Alert when a host behaves differently from peers
Forecast     ← Alert when metric is predicted to breach threshold
```

#### Step 2 — Define the Metric

```
avg:system.cpu.user{env:prod} by {host}
```

- `avg:` — aggregation function
- `system.cpu.user` — the metric
- `{env:prod}` — filter scope
- `by {host}` — group results per host

#### Step 3 — Set Alert Conditions

```yaml
Alert threshold:   > 90   # triggers ALERT
Warning threshold: > 75   # triggers WARN (optional)
Evaluation window:  5 minutes
Recovery:          < 85   # returns to OK
```

#### Step 4 — Configure Notifications

```
Monitor name: [{{env.name}}] High CPU on {{host.name}}
Message:
  CPU usage is {{value}}% on {{host.name}}
  @slack-ops-alerts @pagerduty
  {{#is_recovery}}CPU has recovered to {{value}}%{{/is_recovery}}
```

#### Step 5 — Add Tags & Metadata

```
env:prod
service:web-api
team:platform
priority:P2
```

---

## 4. Monitor Types Deep Dive

### 4.1 Metric Monitor

Best for: CPU, memory, disk, latency, request rates.

```bash
# Example: High Error Rate
avg:trace.web.request.errors{env:prod,service:api} / 
avg:trace.web.request.hits{env:prod,service:api} * 100

Alert if > 5%
```

### 4.2 Log Monitor

Best for: Detecting error patterns in logs.

```
Query: source:nginx status:error
Group by: @http.url
Alert if count > 100 over 5 minutes
```

### 4.3 APM Monitor (Trace)

Best for: P99 latency, error rates from distributed traces.

```
Service: checkout-service
Resource: POST /api/orders
Metric: p99 latency
Alert if > 2000ms
```

### 4.4 Synthetic Monitor

Best for: Uptime and user journey testing.

```
Type:     Browser test / API test
URL:      https://app.example.com/health
Interval: Every 1 minute
Locations: AWS us-east-1, eu-west-1
```

### 4.5 Anomaly Monitor

Best for: Metrics with seasonal patterns.

```
Metric:    aws.rds.database_connections{db:prod}
Algorithm: Agile (adapts to shifts)
Bounds:    2 standard deviations
```

### 4.6 Composite Monitor

Best for: Reducing noise by combining signals.

```
Alert ONLY IF:
  Monitor A (High CPU) = ALERT
  AND
  Monitor B (High Memory) = ALERT
```

---

## 5. Triggered Monitors & Alerting

### Viewing Triggered Monitors

Navigate to: **Monitoring → Triggered**

This shows all monitors currently in ALERT or WARN state.

### Triage Workflow

```
1. Review triggered monitor list
2. Click monitor name → inspect alert details
3. Check metric graph for context
4. Assign to team member (Edit Recipients)
5. Silence if expected (Add Downtime)
6. Investigate root cause
7. Resolve or escalate
```

### Priority Levels

| Priority | Use Case | Response SLA |
|----------|----------|-------------|
| P1 | Service down, data loss | Immediate (< 5 min) |
| P2 | Degraded performance | < 30 minutes |
| P3 | Non-critical warnings | < 2 hours |
| P4 | Informational | Next business day |
| P5 | Low priority / drafts | Best effort |

### Monitor States in Practice

```
[Jenkins] Anomalous Request Volume
  Priority: P5
  Status:   NO DATA   ← Agent may be down or metric not reporting
  Tags:     service:jenkins, env:prod
  State:    DRAFT     ← Not yet active/enabled
```

---

## 6. Downtimes & Maintenance Windows

### Creating a Downtime

Navigate to: **Monitoring → Downtimes → Schedule Downtime**

```yaml
Scope:       env:prod,service:jenkins
Start:       2024-03-15 02:00 UTC
End:         2024-03-15 04:00 UTC
Reason:      Scheduled deployment v2.5.0
Notify:      @slack-ops-channel
Recurrence:  None / Daily / Weekly / Monthly
```

### Downtime Best Practices

- Always add a reason/message for audit trail
- Use scoped tags (`service:X`) rather than silencing all monitors
- Set end time conservatively — extend if needed
- Notify on-call team when downtime starts/ends
- Use `mute_first_recovery_notification: true` to avoid false recoveries

### One-Click Mute from Alert

From a triggered monitor, click **Mute** to silence for:
- 30 minutes / 1 hour / 4 hours / 1 day / custom

---

## 7. SLOs — Service Level Objectives

### What is an SLO?

An SLO defines a target reliability level for a service over a time window.

```
SLO: 99.9% of HTTP requests complete with status < 500
     over a rolling 30-day window
```

### SLO Types

| Type | Best For |
|------|----------|
| **Metric-based** | Error rate, latency percentiles |
| **Monitor-based** | Uptime (good/bad minutes) |
| **Time Slice** | Custom uptime calculations |

### Creating a Metric-Based SLO

```yaml
Name:    API Availability - Production
Type:    Metric
Good events:  sum:trace.web.request.hits{status:2xx OR status:3xx OR status:4xx}
Total events: sum:trace.web.request.hits{}
Target:  99.9%
Window:  30 days
```

### Error Budget

```
Error Budget = (1 - SLO target) × window duration
For 99.9% over 30 days:
  Budget = 0.1% × 43,200 min = 43.2 minutes of downtime allowed
```

### SLO Alerts

Set up burn rate alerts to catch when you're consuming error budget too fast:

```yaml
Fast burn:  14× rate over 1 hour   → Page on-call
Slow burn:  2× rate over 6 hours   → Ticket/Slack alert
```

---

## 8. Event Management

### Triage Inbox

Navigate to: **Monitoring → Event Management → Triage Inbox**

The triage inbox aggregates correlated events into cases for faster investigation.

```
Event Stream → Correlation Rules → Cases → Assignment → Resolution
```

### Event Attributes

```json
{
  "title": "Deployment: checkout-service v2.3.1",
  "text": "Deployed by CI/CD pipeline",
  "tags": ["env:prod", "service:checkout"],
  "alert_type": "info",
  "source_type_name": "jenkins"
}
```

### Sending Custom Events via API

```bash
curl -X POST "https://api.datadoghq.com/api/v1/events" \
  -H "DD-API-KEY: ${DD_API_KEY}" \
  -d '{
    "title": "Deployment completed",
    "text": "Version 2.5.0 deployed to production",
    "tags": ["env:prod", "service:api"],
    "alert_type": "success"
  }'
```

---

## 9. Monitor Quality & Best Practices

### Navigate to: Monitoring → Quality

Datadog scores monitors on:

| Check | Description |
|-------|-------------|
| **Notification configured** | Monitor has at least one alert channel |
| **Has tags** | Monitor is properly tagged for routing |
| **Not always triggering** | Monitor isn't in permanent alert state |
| **Reasonable threshold** | Alert conditions make statistical sense |
| **Has runbook URL** | Links to incident response documentation |

### Common Anti-Patterns to Avoid

```
❌ Alert on every spike — add min evaluation periods
❌ Alert without runbook — always link to docs
❌ Monitor without owner tag — add team:X tag
❌ Overlapping monitors — use composite monitors
❌ Missing recovery threshold — always set recovery
❌ P1 monitors with no PagerDuty — critical alerts must page
```

### Recommended Monitor Template

```
Name:     [Service] [Condition] - [Environment]
Example:  [Checkout] High Error Rate - Production

Tags:
  service: checkout
  env:     prod
  team:    payments
  severity: critical

Message:
  ## Summary
  Error rate is {{value}}% (threshold: {{threshold}}%)
  
  ## Impact
  Customers may be unable to complete purchases
  
  ## Investigation
  Runbook: https://wiki.internal/runbooks/checkout-errors
  Dashboard: https://app.datadoghq.com/dashboard/xxx
  
  ## Contacts
  @pagerduty-payments @slack-payments-oncall
```

---

## 10. Notification Setup

### Navigate to: Monitoring → Settings → Notifications

### Supported Channels

```
Email         → @user@company.com
Slack         → @slack-channel-name
PagerDuty     → @pagerduty-service-name
Opsgenie      → @opsgenie-team
Webhooks      → @webhook-custom
Microsoft Teams → @teams-channel
Jira          → Auto-create tickets
ServiceNow    → Incident creation
```

### Setting Up Slack Integration

1. Go to **Integrations → Slack**
2. Click **Add Channel**
3. Authenticate with your Slack workspace
4. Select the channel (e.g., `#ops-alerts`)
5. Use `@slack-ops-alerts` in monitor notifications

### Notification Template Variables

```
{{host.name}}       ← Hostname that triggered
{{value}}           ← Current metric value
{{threshold}}       ← Alert threshold
{{comparator}}      ← > or <
{{env.name}}        ← Environment tag value
{{service.name}}    ← Service tag value
{{alert_title}}     ← Monitor name
{{last_triggered_at}} ← Timestamp

# Conditional blocks:
{{#is_alert}}   ...alert message... {{/is_alert}}
{{#is_warning}} ...warn message...  {{/is_warning}}
{{#is_recovery}}...recovery msg...  {{/is_recovery}}
{{#is_no_data}} ...no data msg...   {{/is_no_data}}
```

### Next: Set Up Notifications Button

When viewing a newly created monitor, click **"Next, Set Up Notifications"** to configure alert channels without leaving the monitor context.

---

## 11. Monitor Packs & Templates

### What are Monitor Packs?

Monitor Packs are pre-built collections of monitors for common services and infrastructure.

Navigate to: **New Monitor → Browse Templates**

### Available Packs

```
Infrastructure:
  ├── Host (CPU, Memory, Disk, Network)
  ├── Docker / Kubernetes
  ├── AWS (EC2, RDS, ELB, Lambda)
  └── GCP / Azure

Application:
  ├── Jenkins CI/CD        ← Visible in your screenshot
  ├── Nginx / Apache
  ├── PostgreSQL / MySQL
  └── Redis / Kafka

Custom Monitor Packs:
  └── Your organization's pre-built monitors
```

### Deploying a Monitor Pack (Jenkins Example)

From your environment, the following Jenkins monitors are visible:

```
[Jenkins] Anomalous Request Volume  ← P5, DRAFT, NO DATA
[Jenkins] Elevated Error Rate       ← P5, DRAFT, NO DATA

Tags: service:jenkins, env:prod
```

To activate these monitors:
1. Click the monitor name
2. Change status from **DRAFT** to **Active**
3. Configure notification channels
4. Verify the metric is reporting (check for NO DATA issues)

### Export to Terraform

```hcl
# Exported from Datadog Monitor Pack
resource "datadog_monitor" "jenkins_error_rate" {
  name    = "[Jenkins] Elevated Error Rate"
  type    = "metric alert"
  query   = "avg(last_5m):..."
  
  message = "Jenkins error rate is high @pagerduty"
  
  tags = [
    "service:jenkins",
    "env:prod"
  ]
  
  priority = 5
}
```

---

## 12. Hands-On Labs

### Lab 1 — Create a CPU Alert Monitor (15 minutes)

**Objective:** Alert when CPU > 80% for 5 minutes

```
1. Navigate: Monitoring → New Monitor → Metric
2. Metric: avg:system.cpu.user{*} by {host}
3. Alert threshold: > 80
4. Warning threshold: > 70
5. Evaluation: last 5 minutes
6. Name: [Infrastructure] High CPU - {{host.name}}
7. Message: CPU at {{value}}% on {{host.name}}
8. Tag: env:lab, team:workshop
9. Save Monitor
```

**Verify:** Monitor appears in Monitor List with OK or NO DATA status.

---

### Lab 2 — Schedule a Maintenance Downtime (10 minutes)

**Objective:** Silence your Lab 1 monitor for 30 minutes

```
1. Navigate: Monitoring → Downtimes
2. Click: Schedule Downtime
3. Scope: tag:env:lab
4. Duration: 30 minutes from now
5. Message: Workshop lab maintenance
6. Save Downtime
```

**Verify:** Monitor shows "Muted" status in Monitor List.

---

### Lab 3 — Create a Composite Monitor (20 minutes)

**Objective:** Alert only when BOTH CPU and Memory are high

```
Prerequisites: Create a Memory monitor first
  - Metric: avg:system.mem.pct_usable{*} by {host}
  - Alert if < 20% (low usable = high usage)

Steps:
1. Navigate: Monitoring → New Monitor → Composite
2. Select Monitor A: Your CPU monitor
3. Select Monitor B: Your Memory monitor
4. Logic: A && B  (ALERT only when both alert)
5. Name: [Infrastructure] High CPU AND Memory
6. Save
```

**Verify:** Composite monitor is created. It won't alert unless both fire simultaneously.

---

### Lab 4 — Set Up an SLO (20 minutes)

**Objective:** Create a 99.9% availability SLO

```
1. Navigate: Monitoring → SLOs → New SLO
2. Type: Monitor Based
3. Select: Your CPU monitor from Lab 1
4. Good state: OK, WARN
5. Bad state: ALERT, NO DATA
6. Target: 99.9%
7. Window: 7 days (short for lab)
8. Name: Lab - Host CPU Availability
9. Save SLO
```

**Verify:** SLO dashboard shows error budget and current compliance.

---

### Lab 5 — Explore Monitor Quality (10 minutes)

```
1. Navigate: Monitoring → Quality
2. Review your Lab 1 monitor's quality score
3. Identify what's missing (runbook URL, description)
4. Edit your monitor to add:
   - Runbook URL in message
   - Additional tags (team:workshop)
5. Re-check quality score improvement
```

---

## Appendix

### Quick Reference — Monitor API

```bash
# List all monitors
curl "https://api.datadoghq.com/api/v1/monitor" \
  -H "DD-API-KEY: ${DD_API_KEY}" \
  -H "DD-APPLICATION-KEY: ${DD_APP_KEY}"

# Mute a monitor
curl -X POST "https://api.datadoghq.com/api/v1/monitor/{id}/mute" \
  -H "DD-API-KEY: ${DD_API_KEY}"

# Delete a monitor
curl -X DELETE "https://api.datadoghq.com/api/v1/monitor/{id}" \
  -H "DD-API-KEY: ${DD_API_KEY}"
```

### Useful Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Open Datadog search |
| `G M` | Go to Monitors |
| `G D` | Go to Dashboards |

### Glossary

| Term | Definition |
|------|-----------|
| **Agent** | Lightweight software running on hosts to collect metrics |
| **DogStatsD** | Protocol for sending custom metrics to Datadog |
| **Flapping** | Monitor rapidly switching between states |
| **No Data** | No metrics received within evaluation window |
| **Renotify** | Sending repeated alerts while monitor stays triggered |
| **Tag** | Key:value pair for filtering and grouping (`env:prod`) |
| **Watchdog** | Datadog's AI anomaly detection feature |

---

### Workshop Resources

- 📘 Datadog Docs: https://docs.datadoghq.com/monitors/
- 🎓 Datadog Learning Center: https://learn.datadoghq.com
- 🛠️ Terraform Provider: https://registry.terraform.io/providers/DataDog/datadog
- 📊 Monitor SLO Guide: https://docs.datadoghq.com/service_management/service_level_objectives/

---

*Workshop Version 1.0 | Created for Datadog Monitoring Training*
