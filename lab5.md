# Lab 2 — Deep Dive into Traces, Spans & Resource Analysis

**Service:** Jenkins &nbsp;|&nbsp; **Duration:** 1.5 hrs

**Objective:** Learn to read trace data in detail, identify slow resources, understand span metadata, and use the Trace Explorer's analytics capabilities — using Jenkins as the monitored service.

---

## Part 1 — Trace Explorer: Filters and Search

*Estimated time: 25 min*

- Navigate to **APM → Traces**
- Set time range to **Past 1 Hour**
- Clear any existing filters in the search bar

Practice the following searches one at a time (observe results after each):

| Search Query | What to Observe |
|---|---|
| `service:jenkins` | All Jenkins traces |
| `service:jenkins @http.status_code:200` | Successful build/API requests only |
| `service:jenkins @http.status_code:[400 TO 599]` | Client and server errors |
| `service:jenkins @duration:>500ms` | Slow traces over 500ms |
| `service:jenkins @http.method:POST` | POST requests only (e.g., build triggers) |

For the duration filter (`@duration:>500ms`), click one of the returned traces:

- In the **Flame Graph**, identify which span accounts for most of the duration
- Is it Jenkins itself, or a downstream call (e.g., SCM checkout, agent provisioning, plugin call)?

In the **Facets panel** (left side), scroll down and explore available facets:

- HTTP Method
- HTTP Status Code
- Resource Name
- Duration (use the histogram at the top of the facet to see distribution)

Click the **Duration histogram** — drag to select traces between 100ms–500ms. This filters the trace list to medium-latency requests only.

> ✅ **Checkpoint:** How many traces had duration > 500ms in the past hour? Note this number.

---

## Part 2 — Analytics: Aggregate View

*Estimated time: 20 min*

- In the Trace Explorer, click the **Analytics** tab (next to the List tab, looks like a bar chart icon)
- This switches from individual traces to aggregated metrics over traces
- Set the **Measure** dropdown to `Count`; **Group by** to `Resource Name`
- You now see a bar chart of request count per Jenkins endpoint/job

- Change **Measure** to `Duration (p95)`; keep **Group by** as `Resource Name`
- This shows the 95th percentile latency per endpoint
- Identify the slowest endpoint by p95 latency (e.g., `/job/<name>/build`, `/blue/rest/...`)

- Change the visualization type (icons above the chart) from **Bar Chart** to **Timeseries**
- You can now see latency trends over time per Jenkins resource

- Switch **Group by** to `HTTP Status Code`
- Observe the breakdown of 2xx, 4xx, 5xx codes over time

**Export to Dashboard:**

1. Click **Export** (top right of the analytics panel) → **Export to Dashboard**
2. Choose **New Dashboard**
3. Name it: `Jenkins APM Analytics`
4. Click **Save and Open Dashboard**
5. You will expand this dashboard in Lab 3

> ✅ **Checkpoint:** Which resource (endpoint/job) has the highest p95 latency?

---

## Part 3 — Span Details & Tags Deep Dive

*Estimated time: 20 min*

- Return to **APM → Traces**
- Click any trace from your top (slowest) Jenkins resource
- In the **Flame Graph**, click the root span (the widest bar at the top)

In the right panel, review every tag listed under **Metadata**:

- `http.url` — full Jenkins URL requested (e.g., `/job/my-pipeline/build`)
- `http.method` — GET/POST/etc
- `http.status_code` — response code
- `http.useragent` *(if present)* — client type (CI runner, browser, webhook)
- `peer.hostname` or `out.host` — upstream/downstream host (e.g., Git server, agent node)
- `span.kind` — server, client, internal

Scroll down in the Metadata panel to find any custom tags your Jenkins instance may be sending (e.g., job name, build number, agent label).

Click a **child span** (a narrower bar inside the Flame Graph):

- Compare its tags to the root span
- Note: child spans may show SCM operations, plugin calls, agent provisioning steps, or external API calls made by Jenkins pipelines

Click **Full Page** (or open trace in a new tab if available) to get the expanded trace view:

- Switch between **Flame Graph** and **Span List** views
- The Span List view shows a sortable table of all spans — sort by **Duration (descending)**
- The longest span is your bottleneck (e.g., agent startup, workspace checkout, test execution)

> ✅ **Checkpoint:** Take note of the longest child span's name and duration.

---

## Part 4 — Resource Page Exploration

*Estimated time: 25 min*

- Go to **APM → Services** → click your **Jenkins** service
- Click the **Resources** tab
- Sort the resource table by **P95 Latency** (click the column header)
- Click your slowest resource to open the **Resource Detail Page**

On the Resource Detail Page, review:

- **Request Rate** graph
- **Error Rate** graph
- **Latency Distribution** (histogram showing spread of response times)
- **Apdex** for this specific Jenkins resource

Scroll down to the **Span Summary** section:

- This shows which downstream spans are called by this resource (e.g., Git server calls, agent connections, external integrations)
- Each row shows avg duration, % of total time, and call count
- Click **View Traces for this Resource** to see traces filtered to this endpoint

In the Resource Detail Page, click the **Deployments** tab *(if visible)*:

- This shows if latency changed after a recent Jenkins version update or plugin change

---

> 🎉 **Lab 2 Complete.** You can now navigate Jenkins traces analytically, identify slow resources, and read span metadata.
