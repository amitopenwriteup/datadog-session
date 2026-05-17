# Datadog APM Navigation Labs
### Ubuntu Server · Nginx · Datadog Agent with APM
**5 Labs × 1.5 Hours Each | All Steps via Datadog UI**

---

# Lab 1 — Verifying Agent Health & APM Ingestion (1.5 hrs)

**Objective:** Confirm the Datadog Agent is healthy, APM is receiving traces from your Nginx app, and understand the Datadog UI layout before deeper investigation.

---

## Part 1 — Agent Status in the Datadog UI (20 min)

1. Log in to **https://app.datadoghq.com**
2. In the left sidebar, go to **Infrastructure → Infrastructure List**
3. Locate your Ubuntu server hostname in the list
   - The green dot next to it means the Agent is reporting
   - If the dot is grey or red, the Agent is not sending data — check connectivity before proceeding
4. Click the hostname to open the **Host Detail Panel** on the right
5. In the Host Detail Panel, review:
   - **Apps running** — confirm `nginx` appears in the process list
   - **Agent version** — note the version shown; APM requires Agent 6+
   - **Tags** — check that environment tags (e.g., `env:prod`) are present
6. Click **View Host Dashboard** (button at top of the Host Detail Panel)
7. On the Host Dashboard, scroll through the following widgets and note baseline values:
   - CPU Utilization
   - Memory Used
   - I/O Wait
   - Network bytes sent/received

> **Checkpoint:** Screenshot the Host Dashboard. You will compare this baseline in Labs 4 and 5.

---

## Part 2 — Navigating to APM Home (20 min)

1. In the left sidebar, go to **APM → Services**
2. You will land on the **Service Catalog**
   - Locate your Nginx service (it may appear as `nginx` or the name set in `DD_SERVICE`)
   - If no services appear, check that `apm_config.enabled: true` is set — navigate to **Infrastructure → Infrastructure List → your host** and confirm APM is listed under Apps
3. Use the **Environment** dropdown at the top of the Service Catalog to filter by your environment tag (e.g., `env:prod`)
4. Click your Nginx service name to open the **Service Overview Page**
5. On the Service Overview Page, identify and read each section:
   - **Requests** graph — total requests per second
   - **Errors** graph — error rate
   - **Latency** graph — p50, p75, p95, p99 percentiles
   - **Apdex** score (if enabled)
6. Change the time range (top right) to **Past 1 Hour**, then **Past 4 Hours**, and observe how graphs change
7. Click the **Resources** tab below the graphs
   - Resources are individual endpoints/routes (e.g., `GET /`, `POST /api/data`)
   - Note which resource has the highest request count

> **Checkpoint:** Note the p95 latency for your top resource. You will set an alert on this in Lab 3.

---

## Part 3 — Exploring the Trace List (25 min)

1. From the Service Overview Page, click **View Traces** (or go to **APM → Traces** in the sidebar)
2. On the Trace Explorer, set the search filter to your service:
   - In the search bar type: `service:nginx` (replace with your actual service name)
   - Press Enter
3. Observe the **Trace List** — each row is one request end-to-end
4. Click any trace row to open the **Trace Detail Panel**
5. In the Trace Detail Panel, explore:
   - **Flame Graph** — shows each span and its duration as a horizontal bar
   - **Span List** — text list of all spans in the trace
   - **Service Map** (if downstream services exist) — shows call chain
6. Click individual spans in the Flame Graph:
   - Note the **Resource Name** (e.g., `GET /`)
   - Note the **Duration**
   - Note **Metadata / Tags** on the right panel (http.status_code, http.url, etc.)
7. In the Trace Explorer search bar, add a filter for errors:
   - Type: `service:nginx status:error`
   - If no errors appear, remove the status filter and continue
8. Use the **Facets panel** on the left side to filter by:
   - `HTTP Status Code` — click `200` to see only successful requests
   - `Resource Name` — click your top endpoint to isolate it
9. Click **Save View** (top right of Trace Explorer) → name it `Nginx Overview` → Save

> **Checkpoint:** Open 3 different traces and compare their Flame Graphs. Identify which span takes the longest.

---

## Part 4 — Service Map Overview (25 min)

1. In the left sidebar, go to **APM → Service Map**
2. The Service Map renders all instrumented services as nodes connected by arrows
3. Locate your Nginx node
   - Color coding: Green = healthy, Yellow = degraded, Red = high error rate
4. Click the Nginx node:
   - A popup appears with Requests, Errors, and Latency mini-graphs
   - Click **View Service** to jump to the Service Overview Page
5. If your app has downstream dependencies (databases, APIs):
   - Follow the arrows from Nginx to those nodes
   - Click each downstream node and review its metrics
6. Use the **Environment** and **Time** filters at the top of the Service Map to change scope
7. Click **Inspect** on the Nginx node — this shows the inferred service dependencies Datadog detected automatically

> **Lab 1 Complete.** You have verified agent health, explored the Service Catalog, Trace Explorer, and Service Map.

---
---

# Lab 2 — Deep Dive into Traces, Spans & Resource Analysis (1.5 hrs)

**Objective:** Learn to read trace data in detail, identify slow resources, understand span metadata, and use the Trace Explorer's analytics capabilities.

---

## Part 1 — Trace Explorer: Filters and Search (25 min)

1. Navigate to **APM → Traces**
2. Set time range to **Past 1 Hour**
3. Clear any existing filters in the search bar
4. Practice the following searches one at a time (observe results after each):

   | Search Query | What to Observe |
   |---|---|
   | `service:nginx` | All nginx traces |
   | `service:nginx @http.status_code:200` | Successful requests only |
   | `service:nginx @http.status_code:[400 TO 599]` | Client and server errors |
   | `service:nginx @duration:>500ms` | Slow traces over 500ms |
   | `service:nginx @http.method:POST` | POST requests only |

5. For the duration filter (`@duration:>500ms`), click one of the returned traces
   - In the Flame Graph, identify which span accounts for most of the duration
   - Is it Nginx itself, or a downstream call?

6. In the **Facets** panel (left side), scroll down and explore available facets:
   - `HTTP Method`
   - `HTTP Status Code`
   - `Resource Name`
   - `Duration` (use the histogram at the top of the facet to see distribution)

7. Click the **Duration** histogram — drag to select traces between 100ms–500ms
   - This filters the trace list to medium-latency requests only

> **Checkpoint:** How many traces had duration > 500ms in the past hour? Note this number.

---

## Part 2 — Analytics: Aggregate View (20 min)

1. In the Trace Explorer, click the **Analytics** tab (next to the List tab, looks like a bar chart icon)
2. This switches from individual traces to aggregated metrics over traces
3. Set the **Measure** dropdown to `Count`; **Group by** to `Resource Name`
   - You now see a bar chart of request count per endpoint
4. Change **Measure** to `Duration (p95)`; keep **Group by** as `Resource Name`
   - This shows the 95th percentile latency per endpoint
   - Identify the slowest endpoint by p95 latency
5. Change the visualization type (icons above the chart) from Bar Chart to **Timeseries**
   - You can now see latency trends over time per resource
6. Switch **Group by** to `HTTP Status Code`
   - Observe the breakdown of 2xx, 4xx, 5xx codes over time
7. Click **Export** (top right of the analytics panel) → **Export to Dashboard**
   - Choose **New Dashboard**
   - Name it: `Nginx APM Analytics`
   - Click **Save and Open Dashboard**
   - You will expand this dashboard in Lab 3

> **Checkpoint:** Which resource (endpoint) has the highest p95 latency?

---

## Part 3 — Span Details & Tags Deep Dive (20 min)

1. Return to **APM → Traces**
2. Click any trace from your top (slowest) resource
3. In the Flame Graph, click the root span (the widest bar at the top)
4. In the right panel, review every tag listed under **Metadata**:
   - `http.url` — full URL requested
   - `http.method` — GET/POST/etc
   - `http.status_code` — response code
   - `http.useragent` (if present) — client type
   - `peer.hostname` or `out.host` — upstream/downstream host
   - `span.kind` — server, client, internal
5. Scroll down in the Metadata panel to find any **custom tags** your app may be sending
6. Click a child span (a narrower bar inside the Flame Graph)
   - Compare its tags to the root span
   - Note: child spans may show database query strings, cache key names, etc.
7. Click **Full Page** (or open trace in a new tab if available) to get the expanded trace view
8. In the expanded view, switch between **Flame Graph** and **Span List** views
   - The Span List view shows a sortable table of all spans — sort by Duration (descending)
   - The longest span is your bottleneck

> **Checkpoint:** Take note of the longest child span's name and duration.

---

## Part 4 — Resource Page Exploration (25 min)

1. Go to **APM → Services** → click your Nginx service
2. Click the **Resources** tab
3. Sort the resource table by **P95 Latency** (click the column header)
4. Click your slowest resource to open the **Resource Detail Page**
5. On the Resource Detail Page, review:
   - **Request Rate** graph
   - **Error Rate** graph
   - **Latency Distribution** (histogram showing spread of response times)
   - **Apdex** for this specific resource
6. Scroll down to the **Span Summary** section:
   - This shows which downstream spans are called by this resource
   - Each row shows avg duration, % of total time, and call count
7. Click **View Traces for this Resource** to see traces filtered to this endpoint
8. In the Resource Detail Page, click the **Deployments** tab (if visible)
   - This shows if latency changed after a recent deployment

> **Lab 2 Complete.** You can now navigate traces analytically, identify slow resources, and read span metadata.

---
---

# Lab 3 — Monitors, Alerts & Dashboards for APM (1.5 hrs)

**Objective:** Build APM monitors for latency and error rate, create a custom APM dashboard, and configure alert notifications — all through the Datadog UI.

---

## Part 1 — Create an APM Latency Monitor (30 min)

1. In the left sidebar, go to **Monitors → New Monitor**
2. Select **APM** as the monitor type
3. On the monitor configuration page:

   **Step 1 — Define the metric:**
   - Select **APM Metrics** (not Trace Analytics)
   - Service: select your Nginx service
   - Resource: `*` (all resources) or select your slowest resource from Lab 2
   - Metric: `p95 latency`
   - Environment: your env tag (e.g., `prod`)

   **Step 2 — Set alert conditions:**
   - Alert threshold: `500` (ms) — Alert when p95 latency is above 500ms
   - Warning threshold: `300` (ms)
   - Evaluation window: `last 5 minutes`

   **Step 3 — Configure notifications:**
   - Monitor name: `[Nginx] High p95 Latency - {{resource_name}}`
   - Message body (type in the message box):
     ```
     Nginx p95 latency has exceeded threshold.
     Service: {{service.name}}
     Resource: {{resource_name}}
     Current p95: {{value}}ms
     Environment: {{env}}
     Runbook: Check APM traces for slow spans.
     ```
   - In the **Notify your team** section, add your email or a Slack channel if configured

   **Step 4 — Tags:**
   - Add tags: `service:nginx`, `env:prod`, `team:platform`

4. Click **Save** — the monitor is now active

---

## Part 2 — Create an APM Error Rate Monitor (20 min)

1. Go to **Monitors → New Monitor → APM**
2. Configure:
   - Service: your Nginx service
   - Resource: `*`
   - Metric: **Error Rate**
   - Alert threshold: `5` (percent) — alert when error rate > 5%
   - Warning threshold: `2` (percent)
   - Evaluation window: `last 10 minutes`
3. Monitor name: `[Nginx] High Error Rate - {{resource_name}}`
4. Message:
   ```
   Nginx error rate spike detected.
   Current error rate: {{value}}%
   Service: {{service.name}}
   Check Trace Explorer for error traces: 
   https://app.datadoghq.com/apm/traces?query=service:nginx status:error
   ```
5. Click **Save**

---

## Part 3 — View All Monitors & Manage Status (15 min)

1. Go to **Monitors → Manage Monitors**
2. In the search bar, type `nginx` to filter your monitors
3. You will see both monitors you created with status **OK**, **No Data**, or **Alert**
4. Click the latency monitor row to open it
5. On the monitor detail page, explore:
   - **Status & History** tab — shows state changes over time (green/red timeline)
   - **Event Stream** — recent alert/resolve events for this monitor
   - **Related Graphs** — auto-generated graphs of the monitored metric
6. Click **Edit** (pencil icon) and change the evaluation window to **15 minutes** → Save
7. Click the **Mute** button — select **Mute for 1 hour**
   - This is how you silence alerts during maintenance windows
   - Click **Unmute** immediately after to restore it
8. Go back to **Manage Monitors** and use the **Group by** dropdown to group monitors by `service`
   - All your nginx monitors now appear together

> **Checkpoint:** Both monitors should be in OK state if no workload is running.

---

## Part 4 — Build a Custom APM Dashboard (25 min)

1. In the sidebar, go to **Dashboards → New Dashboard**
2. Name it: `Nginx APM Operations` → Click **New Dashboard**
3. You are now in the Dashboard Editor. Add widgets:

   **Widget 1 — Timeseries: Request Rate**
   - Click **Add Widget** → select **Timeseries**
   - In the widget editor, set:
     - Data source: `APM`
     - Service: your Nginx service
     - Metric: `hits` (request count)
     - Display: Line
   - Title: `Request Rate`
   - Click **Save**

   **Widget 2 — Timeseries: Error Rate**
   - Add another Timeseries widget
   - Metric: `errors`
   - Title: `Error Rate`
   - Save

   **Widget 3 — Timeseries: Latency Percentiles**
   - Add Timeseries widget
   - Add 3 queries (click + Add Query for each):
     - Query A: Metric `p50 latency`
     - Query B: Metric `p75 latency`
     - Query C: Metric `p95 latency`
   - Title: `Latency Percentiles (p50/p75/p95)`
   - Save

   **Widget 4 — Top List: Slowest Resources**
   - Click **Add Widget** → select **Top List**
   - Data source: `APM`
   - Metric: `p95 latency`
   - Group by: `Resource`
   - Title: `Top 10 Slowest Resources`
   - Save

   **Widget 5 — Monitor Summary**
   - Click **Add Widget** → select **Monitor Summary**
   - Filter: `service:nginx`
   - Title: `Nginx Monitor Status`
   - Save

4. Drag and resize widgets to arrange the dashboard logically (graphs on top, lists below)
5. Click **Save Changes**
6. Click the **Share** icon (top right) → **Generate Public URL** (optional, for sharing with team)

> **Lab 3 Complete.** You have created two APM monitors and a fully custom APM dashboard.

---
---

# Lab 4 — Running Workloads & Observing APM Detection Live (1.5 hrs)

**Objective:** Generate real HTTP workload against your Nginx app and watch Datadog APM detect, trace, and visualize that activity in real time — all observed through the UI.

---

## Part 1 — Establishing Pre-Workload Baseline (15 min)

1. Open your `Nginx APM Operations` dashboard (from Lab 3)
2. Set time range to **Past 30 Minutes**
3. Enable **Auto-Refresh** (clock icon, top right) → set to **Every 30 seconds**
4. Note the current values for all widgets (screenshot or write down):
   - Request rate (req/s)
   - Error rate (%)
   - p95 latency (ms)
   - Which resources are in the Top List

5. Open a second browser tab to **APM → Traces**
   - Filter: `service:nginx`
   - Note the trace volume per minute shown in the histogram at the top of the trace list

> **Baseline is established. Keep both tabs open side-by-side if possible.**

---

## Part 2 — Generate Normal Workload (send HTTP requests to Nginx) (20 min)

> You will generate traffic using any available HTTP tool on your Ubuntu server or local machine. The method you use is outside Datadog — watch Datadog react to it.

**From your Ubuntu server or any machine that can reach Nginx, generate continuous HTTP requests:**

- Use `curl`, a browser, `Apache Bench (ab)`, `wrk`, or `hey`
- Example using curl loop from the server terminal (run this in a shell):
  ```bash
  while true; do curl -s http://localhost/ > /dev/null; sleep 0.2; done
  ```
- If you have `ab` available:
  ```bash
  ab -n 1000 -c 10 http://localhost/
  ```
- If you have multiple endpoints, hit them in rotation

> Note: The exact tool doesn't matter — the goal is to generate HTTP traffic and watch Datadog detect it.

**While the workload runs, observe in Datadog:**

1. On the **Nginx APM Operations** dashboard (auto-refreshing):
   - Watch the **Request Rate** widget spike upward in real time
   - Watch the **Latency Percentiles** graph — do p95/p99 stay stable?
   - The **Top 10 Slowest Resources** list may reorder as more data arrives

2. On the **APM → Traces** tab:
   - The trace histogram at the top of the list shows new traces appearing
   - Click **Live Tail** (toggle, top right of Trace Explorer) — traces stream in real time as requests arrive
   - Watch trace rows appear with their resource names, durations, and status codes

3. On **APM → Services → your Nginx service**:
   - The Request Rate graph climbs
   - p50 and p95 latency graphs update

> **Checkpoint:** Note the new request rate (req/s) during workload. Compare to the baseline from Part 1.

---

## Part 3 — Generate Error Workload (15 min)

Now generate requests that will cause HTTP errors (404s, 500s) to trigger the error rate monitor.

**Generate 404 errors:**
```bash
while true; do curl -s http://localhost/nonexistent-page-xyz > /dev/null; sleep 0.3; done
```

**Generate mixed traffic (good + bad) in separate terminals:**
- Terminal 1: Normal requests to `/`
- Terminal 2: Requests to a non-existent path

**Observe in Datadog:**

1. On the dashboard, watch the **Error Rate** widget — it should start climbing
2. Go to **APM → Traces**:
   - In the search bar, type: `service:nginx status:error`
   - You should now see error traces appearing
   - Click one error trace → in the Flame Graph, look at the root span's `http.status_code` tag
   - Confirm it is `404`
3. Go to **Monitors → Manage Monitors**:
   - Watch the Error Rate monitor status — if error rate crosses 2%, it goes to **Warn**; above 5%, it goes to **Alert**
   - The status dot changes from green → yellow → red
   - If an alert triggers, check your email or Slack for the notification

> **Checkpoint:** Did the Error Rate monitor change status? What was the peak error rate?

---

## Part 4 — Observe in Live Infrastructure View (20 min)

1. Go to **Infrastructure → Infrastructure List**
2. Click your Ubuntu server hostname
3. In the **Host Detail Panel**, switch to the **Metrics** tab
4. While the workload is still running, observe:
   - **CPU %** — Nginx handling requests will increase CPU
   - **Network bytes received/sent** — more traffic = more bytes
5. Click **View Host Dashboard**
   - Compare CPU and network metrics to the baseline you recorded in Lab 1
6. Scroll down to the **Processes** widget on the Host Dashboard
   - You should see `nginx` processes with elevated CPU usage
7. Return to **APM → Service Map**
   - Your Nginx node may have changed color if latency or errors increased
   - Click the Nginx node → the mini-graphs now show live workload data
8. In the Service Map, click **Inspect** on the Nginx node
   - Review which inferred dependencies Datadog detected during the workload

> **Checkpoint:** Note what color your Nginx node is in the Service Map. What does that indicate?

---

## Part 5 — Stop Workload and Observe Recovery (10 min)

1. Stop the traffic generation (Ctrl+C in your terminal)
2. Watch the Dashboard auto-refresh every 30 seconds:
   - Request Rate drops back toward baseline
   - Error Rate returns to 0%
   - p95 latency stabilizes
3. Go to **Monitors → Manage Monitors**:
   - The Error Rate monitor resolves back to **OK**
   - A resolve notification is sent (check email/Slack)
4. Go to **APM → Traces → Live Tail**:
   - Trace stream slows to a trickle as traffic stops

> **Lab 4 Complete.** You have generated real workload and observed Datadog APM detect, surface, and alert on it entirely through the UI.

---
---

# Lab 5 — Advanced APM: Continuous Profiler, SLOs & Incident Navigation (1.5 hrs)

**Objective:** Use Datadog's advanced APM features — Continuous Profiler, SLO management, and the Datadog Notebooks — to analyze performance in depth and document findings.

---

## Part 1 — Continuous Profiler (20 min)

> The Continuous Profiler requires it to be enabled on the Agent side. If not enabled, this section is an exploration of the UI and what you would see.

1. In the left sidebar, go to **APM → Continuous Profiler** (also called **Profile Search**)
2. If profiling data is available, filter by your service name
3. Explore the **Profile List** — each row is a 60-second profiling snapshot
4. Click a profile to open the **Flame Graph Profiler**
   - This shows CPU time broken down by method/function
   - Hover over functions to see % of total CPU time
5. Use the **Profile Type** dropdown to switch between:
   - **CPU** — which functions consume CPU
   - **Wall Time** — real elapsed time
   - **Heap Allocations** — memory allocation hotspots (for supported runtimes)
6. In the Flame Graph, click any wide function bar to zoom in
   - Zooming reveals child functions called by that function
7. Use the search bar inside the profiler to search for `nginx` or any function name
8. Click **Compare** (if available) to compare two profiles side-by-side
   - Select a profile from before the workload and one during it
   - Observe which functions grew in CPU time during load

> If profiling is not enabled, review the documentation linked on the Continuous Profiler page and note what would need to be enabled on the Agent.

---

## Part 2 — Create an SLO (Service Level Objective) (25 min)

1. In the left sidebar, go to **Service Mgmt → SLOs** (or search "SLO" in the nav)
2. Click **New SLO**
3. Select **Monitor Based SLO**
4. On the SLO configuration page:

   **Step 1 — Select monitors:**
   - Click **Add Monitor**
   - Search for and add your `[Nginx] High p95 Latency` monitor from Lab 3
   - Search for and add your `[Nginx] High Error Rate` monitor from Lab 3

   **Step 2 — Set targets:**
   - SLO Target: `99.5%` uptime
   - Time window: **Rolling 30 days**
   - Add a secondary target: `99.9%` for **Rolling 7 days**

   **Step 3 — Name and tags:**
   - SLO Name: `Nginx Service Reliability SLO`
   - Description: `Tracks p95 latency and error rate for the Nginx APM service.`
   - Tags: `service:nginx`, `env:prod`, `team:platform`

5. Click **Save and Set Alert**
6. On the SLO Alert configuration page:
   - Alert type: **Error Budget Alert**
   - Alert when: `50%` of error budget has been consumed
   - Warning when: `25%` consumed
   - Notification message:
     ```
     Nginx SLO error budget is being consumed.
     SLO: {{slo.name}}
     Remaining budget: {{error_budget_remaining}}%
     Window: {{timeframe}}
     ```
7. Click **Save**

8. Return to **Service Mgmt → SLOs** and click your new SLO
9. On the SLO Detail Page, explore:
   - **Status** — current compliance %
   - **Error Budget** — how much budget remains (shown as a bar)
   - **History** — compliance trend over time
   - **Alerts** tab — the alert you just created

> **Checkpoint:** What is your SLO's current compliance percentage?

---

## Part 3 — Event Stream & Correlation (15 min)

1. In the left sidebar, go to **Events → Explorer**
2. In the search bar, type: `source:datadog` and your hostname
3. This shows all events Datadog generated for your server:
   - Agent start/stop events
   - Monitor alert/resolve events
   - Deployment markers (if any)
4. Find a monitor alert event from Lab 4's error workload (if triggered)
   - Click it to expand — it shows the exact time, monitor name, and current value
5. Click **View in Monitor** link inside the event — this jumps directly to the monitor
6. Return to **Events → Explorer**
7. Change the search to: `source:nginx` (if Nginx log events are flowing)
   - If Nginx logs are integrated, you will see HTTP error events here
8. Click the **Correlate** button on any event (if available)
   - Datadog will show APM traces and metrics that occurred at the same time as the event
   - This is how you correlate an alert with its root cause trace

---

## Part 4 — Notebooks: Document Your Findings (20 min)

Datadog Notebooks are live investigation documents that embed real metrics, traces, and events alongside text.

1. In the left sidebar, go to **Notebooks → New Notebook**
2. Title the notebook: `Nginx APM Lab Investigation — [Today's Date]`
3. In the first text cell, write:
   ```
   ## Executive Summary
   This notebook documents findings from the Datadog APM lab session.
   Server: Ubuntu + Nginx
   Environment: prod
   Date: [today]
   ```
4. Click the `+` button below the text cell → **Add Graph**
   - Add your Request Rate timeseries (same query as Lab 3 Dashboard Widget 1)
   - Set the time range to cover the workload period from Lab 4
5. Add another text cell below the graph:
   ```
   ## Workload Observation
   During load generation, request rate increased from [X] to [Y] req/s.
   p95 latency peaked at [Z]ms.
   Error rate peaked at [W]%.
   ```
6. Click `+` → **Add Graph** → add the Latency Percentiles graph
7. Click `+` → **Add Log Stream** (if logs are connected) → filter `source:nginx`
8. Click `+` → **Add Monitor Status** → select your Nginx monitors
9. At the bottom, add a final text cell:
   ```
   ## Recommendations
   1. Set alert threshold at p95 > 300ms to catch degradation earlier.
   2. Investigate [slowest resource] for optimization.
   3. Enable Continuous Profiler for deeper CPU analysis.
   ```
10. Click **Save Notebook**
11. Click **Share** (top right) → copy the link → this notebook can be shared with your team

---

## Part 5 — Wrapping Up: APM Settings Review (10 min)

1. Go to **APM → Settings** (or **Organization Settings → APM**)
2. Review the **Ingestion Control** page:
   - Shows current trace ingestion rate vs. your plan limit
   - Shows which services are sending the most traces
   - If any service exceeds quota, you can set a sampling rate here from the UI
3. Click **Retention Filters** tab:
   - Shows which traces are being retained for 15 days vs. discarded
   - By default, all error traces and all sampled traces are retained
   - Click **Add Filter** to see how you would create a custom retention rule (do not save)
4. Click **Remote Configuration** tab (if available):
   - Shows Agent configuration you can push remotely without SSH access
5. Go to **APM → Services** one final time
   - Your Nginx service card now shows the full history of all your lab work

> **Lab 5 Complete. All 5 Labs Complete.**

---

---

# Summary Reference Card

| Lab | Focus | Key Screens |
|---|---|---|
| Lab 1 | Agent health, Trace Explorer, Service Map | Infrastructure List · APM Services · APM Traces · Service Map |
| Lab 2 | Trace analytics, span metadata, resource analysis | Trace Explorer Analytics · Span Detail · Resource Page |
| Lab 3 | Monitors, alerts, custom dashboard | New Monitor · Manage Monitors · Dashboard Editor |
| Lab 4 | Live workload, real-time APM detection | Live Tail · Host Dashboard · Service Map · Monitor Status |
| Lab 5 | Profiler, SLOs, Notebooks, settings | Continuous Profiler · SLO Manager · Notebooks · APM Settings |

---

## Key Datadog Navigation Paths

```
APM Traces         →  APM → Traces
Service Overview   →  APM → Services → [service name]
Service Map        →  APM → Service Map
Continuous Profiler→  APM → Continuous Profiler
Monitors           →  Monitors → Manage Monitors
New Monitor        →  Monitors → New Monitor
Dashboards         →  Dashboards → Dashboard List
SLOs               →  Service Mgmt → SLOs
Notebooks          →  Notebooks → New Notebook
Infrastructure     →  Infrastructure → Infrastructure List
Event Explorer     →  Events → Explorer
APM Settings       →  APM → Settings
```

---

*All labs performed through Datadog UI navigation only — no scripts, no automation, no CLI.*
