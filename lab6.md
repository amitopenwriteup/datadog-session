# Lab 3 — Dashboards, Monitors & SLOs

**Service:** Jenkins | **Duration:** 2 hrs

**Objective:** Build a production-grade Jenkins APM dashboard, create threshold and anomaly-based monitors, and define a Service Level Objective (SLO) to track Jenkins reliability over time.

**Prerequisite:** Lab 2 completed. The dashboard named `Jenkins APM Analytics` should already exist in your Datadog account from the export step in Lab 2 Part 2.

---

## Part 1 — Expand the Jenkins APM Analytics Dashboard

*Estimated time: 30 min*

- Navigate to **Dashboards** and open `Jenkins APM Analytics`
- Click **Edit** (top right)

You will add widgets in four sections. Use the **Add Widget** button for each.

---

### Section 1 — Header Row (Summary Metrics)

Add three Query Value widgets side by side across the top of the dashboard.

**Widget 1 — Request Rate**

- Widget type: Query Value
- Metric: `trace.servlet.request.hits` (or `trace.http.request.hits`)
- Filter: `service:jenkins`
- Aggregation: `sum`
- Title: `Jenkins Request Rate`
- Unit: `requests/s`

**Widget 2 — Error Rate**

- Widget type: Query Value
- Metric: `trace.servlet.request.errors` divided by `trace.servlet.request.hits`
- Filter: `service:jenkins`
- Title: `Jenkins Error Rate`
- Conditional formatting:
  - Green when value < 0.01
  - Yellow when value >= 0.01 and < 0.05
  - Red when value >= 0.05

**Widget 3 — p95 Latency**

- Widget type: Query Value
- Metric: `trace.servlet.request.duration.by.service.95p`
- Filter: `service:jenkins`
- Title: `Jenkins p95 Latency`
- Unit: `ms`
- Conditional formatting:
  - Green when value < 500
  - Yellow when value >= 500 and < 2000
  - Red when value >= 2000

---

### Section 2 — Latency Timeseries Row

Add two Timeseries widgets.

**Widget 4 — Latency Percentiles Over Time**

- Widget type: Timeseries
- Add three metrics on the same graph:
  - `trace.servlet.request.duration.by.service.50p` — Label: `p50`
  - `trace.servlet.request.duration.by.service.95p` — Label: `p95`
  - `trace.servlet.request.duration.by.service.99p` — Label: `p99`
- Filter all three: `service:jenkins`
- Title: `Jenkins Latency Percentiles`
- Y-axis: `ms`

**Widget 5 — Latency by Resource (Top 5)**

- Widget type: Timeseries
- Metric: `trace.servlet.request.duration.by.resource_service.95p`
- Filter: `service:jenkins`
- Group by: `resource_name`
- Limit to top 5
- Title: `p95 Latency by Resource`

---

### Section 3 — Traffic and Errors Row

Add two more widgets.

**Widget 6 — Request Volume by Resource**

- Widget type: Top List
- Metric: `trace.servlet.request.hits`
- Filter: `service:jenkins`
- Group by: `resource_name`
- Sort: descending
- Title: `Top Jenkins Resources by Request Volume`

**Widget 7 — Error Count by HTTP Status Code**

- Widget type: Timeseries
- Metric: `trace.servlet.request.errors`
- Filter: `service:jenkins`
- Group by: `http.status_code`
- Title: `Jenkins Errors by Status Code`
- Display as: Bars

---

### Section 4 — Span-Level Detail

**Widget 8 — Trace List (Live Feed)**

- Widget type: Trace Stream
- Filter: `service:jenkins`
- Columns to show: `Duration`, `Resource`, `Status`, `HTTP Status Code`
- Title: `Jenkins Live Trace Feed`

---

### Finishing the Dashboard

- Arrange widgets:
  - Row 1: Widgets 1, 2, 3 (summary row)
  - Row 2: Widgets 4, 5 (latency timeseries)
  - Row 3: Widgets 6, 7 (traffic and errors)
  - Row 4: Widget 8 (trace stream, full width)
- Click the dashboard title to rename if needed
- Click **Save Changes**

> Checkpoint: The dashboard should have 8 widgets across 4 rows with no empty gaps. All widgets should show data from the Jenkins service.

---

## Part 2 — Create APM-Based Monitors

*Estimated time: 25 min*

You will create four monitors that cover the core health signals for Jenkins.

---

### Monitor 1 — High p95 Latency (Threshold)

- Go to **Monitors -> New Monitor**
- Select **APM**
- Select **APM Metrics**

Configure:

- Service: `jenkins`
- Resource: `All Resources`
- Metric: `p95 latency`
- Alert threshold: `2000` ms
- Warning threshold: `1000` ms
- Evaluation window: `5 minutes`

Notification message:

```
Jenkins p95 latency is above threshold.

Current value: {{value}} ms
Threshold: 2000 ms

Review the slowest resources in the Jenkins APM Analytics dashboard.
Investigate using APM -> Services -> Jenkins -> Resources (sort by p95).
```

- Monitor name: `[Jenkins] High p95 Latency`
- Tags: `service:jenkins`, `env:prod`, `team:platform`
- Click **Create**

---

### Monitor 2 — Elevated Error Rate (Threshold)

- Go to **Monitors -> New Monitor -> APM -> APM Metrics**

Configure:

- Service: `jenkins`
- Resource: `All Resources`
- Metric: `Error Rate`
- Alert threshold: `5` percent
- Warning threshold: `2` percent
- Evaluation window: `5 minutes`

Notification message:

```
Jenkins error rate has exceeded the threshold.

Current error rate: {{value}}%
Alert threshold: 5%

Check the following:
- APM -> Traces -> filter service:jenkins @http.status_code:[500 TO 599]
- Review Jenkins system logs for plugin failures or JVM errors
- Check if a deployment or config change was recently made
```

- Monitor name: `[Jenkins] Elevated Error Rate`
- Tags: `service:jenkins`, `env:prod`, `team:platform`
- Click **Create**

---

### Monitor 3 — Anomaly Detection on Request Volume

This monitor detects abnormal drops or spikes in Jenkins traffic without requiring a fixed threshold.

- Go to **Monitors -> New Monitor -> Metric**

Configure:

- Metric: `trace.servlet.request.hits`
- Filter: `service:jenkins`
- Aggregation: `sum`
- Evaluation window: `last 1 hour`
- Detection method: **Anomaly**
- Algorithm: `Agile` (handles day-over-day seasonality)
- Deviations: `2`
- Alert when anomaly persists for: `10 minutes`

Notification message:

```
Jenkins request volume is behaving anomalously.

This may indicate:
- A drop in traffic (Jenkins unreachable, CI pipeline paused)
- A spike in traffic (build queue flooding, webhook storm)

Current value: {{value}} requests/s
Expected range: {{threshold}} requests/s

Check Jenkins system status and CI trigger configurations.
```

- Monitor name: `[Jenkins] Anomalous Request Volume`
- Tags: `service:jenkins`, `env:prod`, `team:platform`
- Click **Create**

---

### Monitor 4 — Slow Specific Resource (Per Resource Alert)

This monitor tracks a specific slow endpoint identified in Lab 2 rather than the whole service.

- Go to **Monitors -> New Monitor -> APM -> APM Metrics**

Configure:

- Service: `jenkins`
- Resource: Select the slowest resource you noted in Lab 2 (e.g., `/job/<name>/build` or `/blue/rest/...`)
- Metric: `p95 latency`
- Alert threshold: `3000` ms
- Warning threshold: `1500` ms
- Evaluation window: `10 minutes`

Notification message:

```
The Jenkins resource {{resource_name}} is experiencing high latency.

Current p95: {{value}} ms
Threshold: 3000 ms

Steps to investigate:
1. Open APM -> Traces, filter: service:jenkins resource_name:<name> @duration:>1s
2. Inspect flame graphs for the dominant span
3. Check if agent provisioning, SCM checkout, or plugin call is the bottleneck
```

- Monitor name: `[Jenkins] Slow Resource — {{resource_name}}`
- Tags: `service:jenkins`, `env:prod`, `team:platform`
- Click **Create**

> Checkpoint: Go to Monitors -> Manage Monitors. You should see all four monitors. Confirm their status is OK or No Data (not in an error state due to misconfiguration).

---

## Part 3 — Define a Service Level Objective (SLO)

*Estimated time: 20 min*

An SLO lets you formally define and track a reliability target for Jenkins. You will create two SLOs: one for latency and one for availability.

---

### SLO 1 — Latency SLO (APM-Based)

- Go to **Service Management -> SLOs -> New SLO**
- Select **APM** as the SLO type

Configure:

- Service: `jenkins`
- Metric: `p95 latency`
- Good events definition: requests where p95 latency < 2000 ms
- Target: `95%` of requests within the latency budget
- Time window: `7 days` rolling
- Name: `Jenkins p95 Latency SLO`
- Description: `95% of Jenkins requests must complete with p95 latency under 2 seconds over a 7-day window`
- Tags: `service:jenkins`, `team:platform`

Click **Create**.

---

### SLO 2 — Availability SLO (Monitor-Based)

This SLO uses the error rate monitor you created in Part 2 as its data source.

- Go to **Service Management -> SLOs -> New SLO**
- Select **Monitor** as the SLO type

Configure:

- Select monitor: `[Jenkins] Elevated Error Rate`
- Good state: monitor is in `OK` state
- Target: `99%` uptime
- Time window: `30 days` rolling
- Name: `Jenkins Availability SLO`
- Description: `Jenkins error rate must remain below 5% for 99% of the time over a 30-day rolling window`
- Tags: `service:jenkins`, `team:platform`

Click **Create**.

---

### View SLO Status

- Go to **Service Management -> SLOs**
- Both SLOs should appear
- Click each SLO to review:
  - Current compliance percentage
  - Remaining error budget (how much room is left before the target is breached)
  - Historical burn rate graph

> Checkpoint: Both SLOs should show a compliance percentage. If you see no data, confirm the monitors are correctly configured and the Jenkins service is sending traces.

---

### Add SLO Widgets to the Dashboard

- Go back to **Dashboards -> Jenkins APM Analytics**
- Click **Edit**
- Add two SLO widgets at the bottom of the dashboard

**Widget 9 — Latency SLO Status**

- Widget type: SLO
- Select: `Jenkins p95 Latency SLO`
- Display: Current status + time remaining
- Title: `Latency SLO — 7 Day`

**Widget 10 — Availability SLO Status**

- Widget type: SLO
- Select: `Jenkins Availability SLO`
- Display: Current status + error budget remaining
- Title: `Availability SLO — 30 Day`

- Click **Save Changes**

---

## Part 4 — Monitor Notification Routing (Optional, 10 min)

If your organization has a notification channel configured (Slack, PagerDuty, email), add it to your monitors.

- Open each monitor -> click **Edit**
- In the **Notify your team** section, add:
  - `@slack-jenkins-alerts` (if Slack is connected)
  - or `@pagerduty-jenkins` (if PagerDuty is connected)
  - or enter an email address directly
- Under **Re-notification**, enable re-alert every `30 minutes` while the monitor remains in alert state
- Click **Save**

If no notification channel is configured, skip this part. The monitors will still fire and be visible in the Datadog UI.

---

## Validation Checklist

- [ ] Dashboard `Jenkins APM Analytics` has 10 widgets across 5 rows
- [ ] All three summary widgets (Request Rate, Error Rate, p95 Latency) show data
- [ ] Latency percentile timeseries shows p50, p95, and p99 lines
- [ ] Top List shows resource names ranked by request volume
- [ ] Monitor `[Jenkins] High p95 Latency` exists and is in OK or No Data state
- [ ] Monitor `[Jenkins] Elevated Error Rate` exists and is in OK or No Data state
- [ ] Monitor `[Jenkins] Anomalous Request Volume` exists with anomaly detection enabled
- [ ] Monitor `[Jenkins] Slow Resource` exists targeting the specific slow endpoint from Lab 2
- [ ] SLO `Jenkins p95 Latency SLO` shows a compliance percentage
- [ ] SLO `Jenkins Availability SLO` shows a compliance percentage and error budget
- [ ] Both SLO widgets are added to the dashboard

---

## Key Concepts Recap

**Dashboard:** A collection of widgets that visualize metrics, traces, and logs in a single view. Used for ongoing operational awareness.

**Monitor:** A rule that evaluates a metric or trace condition on a schedule and fires an alert when a threshold or anomaly condition is met.

**Threshold Monitor:** Fires when a value crosses a fixed number (e.g., p95 > 2000ms). Simple and predictable.

**Anomaly Monitor:** Fires when a value deviates from its historical pattern. Useful when normal traffic volume varies by time of day or day of week.

**SLO (Service Level Objective):** A formal reliability target expressed as a percentage over a time window. Backed by an error budget — the acceptable amount of bad events before the target is breached.

**Error Budget:** The allowance for failures built into an SLO. If your availability SLO is 99% over 30 days, your error budget is 1% of 30 days = approximately 7.2 hours of allowed downtime.

---

## Troubleshooting

**Widget shows no data**
- Verify the metric name is correct for your Datadog agent version. In some environments `trace.web.request` is used instead of `trace.servlet.request`. Go to Metrics -> Explorer and search for `trace.` to find the correct metric names available in your account.
- Confirm the `service:jenkins` tag filter matches exactly what Jenkins is reporting. Check APM -> Services to see the exact service name.

**Monitor stuck in No Data**
- APM metric monitors require traces to be flowing. If Jenkins is idle, there will be no data. Trigger a build in Jenkins to generate trace data, then re-evaluate.
- For anomaly monitors, Datadog needs at least a few hours of historical data to establish a baseline. No Data is expected immediately after creation.

**SLO shows 0% or error**
- For monitor-based SLOs, the monitor must have been active and in a known state for at least some portion of the SLO window. If the monitor was just created, the SLO will show incomplete data for the first few hours.
- For APM-based SLOs, confirm the service name and metric are correct.

**Dashboard widgets not rendering**
- Click the three-dot menu on any broken widget and select Edit to review the configuration.
- Check that the time range on the dashboard (top right) is set to a window where data exists (Past 1 Hour or Past 4 Hours).

---

> Lab 3 Complete. You now have a production-grade Jenkins observability setup: a multi-section dashboard, four monitors covering latency, errors, and anomalies, and two SLOs tracking latency and availability with error budgets.

**Up next — Lab 4:** CI Visibility — Jenkins pipeline analytics using Datadog's native CI module, including pipeline execution traces, test result tracking, and flaky test detection.
