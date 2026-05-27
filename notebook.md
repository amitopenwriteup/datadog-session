# Datadog Notebooks — Beginner Workshop

**Duration:** ~90 minutes 
**Level:** Newbie — no prior Datadog experience needed 
**Goal:** By the end of this workshop, you'll know what Datadog Notebooks are, why they matter, and how to build one from scratch.

---

## Table of Contents

1. [What is Datadog?](#1-what-is-datadog)
2. [What is a Notebook?](#2-what-is-a-notebook)
3. [Why Use Notebooks?](#3-why-use-notebooks)
4. [Anatomy of a Notebook](#4-anatomy-of-a-notebook)
5. [Hands-On: Create Your First Notebook](#5-hands-on-create-your-first-notebook)
6. [Working with Cells](#6-working-with-cells)
7. [Time Controls & Template Variables](#7-time-controls--template-variables)
8. [Collaboration Features](#8-collaboration-features)
9. [Common Use Cases](#9-common-use-cases)
10. [Practice Exercises](#10-practice-exercises)
11. [Quiz](#11-quiz)
12. [Cheat Sheet](#12-cheat-sheet)

---

## 1. What is Datadog?

Datadog is a **cloud monitoring and observability platform**. It helps engineering teams answer questions like:

- Is my application running normally right now?
- Why did my service slow down at 3 AM last Tuesday?
- Are users experiencing errors? Where?

Datadog collects **metrics** (numbers over time), **logs** (text records of events), **traces** (journey of a request through your system), and much more — and visualizes all of it in one place.

> **Think of it this way:** If your application is a car, Datadog is the dashboard — it shows you speed, fuel, engine temperature, and any warning lights, all in real time.

---

## 2. What is a Notebook?

A **Datadog Notebook** is a live, collaborative document that mixes:

- **Timeseries and data visualizations** (live metrics, logs, and traces from Datadog)
- **Text and Markdown** (your explanations, instructions, analysis)
- **SQL queries, tables, and data source widgets**

Unlike a static document (like a Word file), a Notebook is **connected to live data**. The graphs update in real time, and you can change the time range to look at different windows.

> **Think of it this way:** A Notebook is like a Google Doc that has charts embedded in it — except those charts are pulling live data from your production systems.

---

## 3. Why Use Notebooks?

Here's why teams love Notebooks:

| Situation | Without Notebooks | With Notebooks |
|---|---|---|
| Something broke at 2 AM | Screenshots in Slack, context lost | Live doc with data + narrative, shared with the team |
| Writing a postmortem | Copy-paste graphs into a doc, goes stale | Embedded live graphs that show exactly what happened |
| New teammate needs a runbook | Plain text doc with no data | Step-by-step guide with live health checks embedded |
| Weekly performance review | Manual report building | Reusable notebook that refreshes automatically |

---

## 4. Anatomy of a Notebook

A Notebook is made up of **cells**. Each cell is either a data visualization or a text block. Here's what each type does:

### 4.1 Timeseries Cell

Plots a metric query over time — the most common cell type for monitoring data.

```
+-------------------------------------+
|  Timeseries Cell                    |
|                                     |
|  Query: avg:system.cpu.user{*}      |
|                                     |
|  [line chart renders here]          |
+-------------------------------------+
```

You write a **metric query**, choose a visualization (line, bar, area), and the cell displays live data.

### 4.2 Text Cell

Plain text or Markdown for headings, explanations, runbook steps, and checklists. Type your content directly — it renders as formatted text when you click away.

```
+-------------------------------------+
|  Text Cell                          |
|                                     |
|  ## Step 2: Check CPU Usage         |
|  Look for spikes above 80%.         |
|  If sustained > 5 min, escalate.    |
+-------------------------------------+
```

### 4.3 Data Source Cell

Connects to a specific Datadog data source — logs, APM traces, RUM events, and more — and displays results in the notebook.

### 4.4 SQL Cell

Lets you write SQL queries against your data and display results as a table or visualization inside the notebook.

### 4.5 Table Cell

Displays structured tabular data. Useful for showing lists of hosts, services, or events side by side with their metrics.

### 4.6 Ask Bits (AI Assistant)

**Ask Bits** is Datadog's built-in AI assistant. You can use it directly inside a notebook to generate queries, explain graphs, or summarize findings in plain language. It appears as the first option in the "Get started with" quick-start bar.

### 4.7 The Toolbar

At the top of every Notebook:

| Control | What It Does |
|---|---|
| **1h / Past 1 Hour** | Time range selector — sets the window for all unlocked cells |
| **UTC offset** | Shows your local timezone (e.g., UTC+05:30) |
| **Playback controls** | Step backward/forward through time or pause live updates |
| **Edit (pencil)** | Switch between view and edit mode |
| **Share** | Generates a shareable URL (blue button, top right) |
| **Three-dot menu (...)** | Access notebook settings, clone, delete, and more |

---

## 5. Hands-On: Create Your First Notebook

> **Estimated time: 10 minutes**

### Step 1 — Open Notebooks

1. Log into your Datadog account at [app.datadoghq.com](https://app.datadoghq.com)
2. In the left sidebar, look for **Notebooks** (usually under the Dashboards section)
3. Click **Notebooks**

### Step 2 — Create a New Notebook

1. Click the **+ New Notebook** button (top right)
2. You'll see a blank notebook with a title field at the top
3. Click on the title and type: `My First Notebook`

### Step 3 — Add a Text Cell

1. Click inside the notebook body where it says `Type / for commands`
2. Type `/` — a command menu will appear
3. Select **Text** from the menu
4. Type the following:

```markdown
## Investigation Start

This notebook tracks the health of our application.
Created: [today's date]
Author: [your name]
```

5. Click outside the cell (or press `Escape`) to exit edit mode

Alternatively, use the **"Get started with"** quick-start bar that appears in a new notebook and click **Timeseries**, **Data Source**, **SQL**, or **Table** directly.

### Step 4 — Add a Timeseries Cell

1. Click in the notebook body and type `/`
2. Choose **Timeseries** from the menu (or click the **Timeseries** button in the quick-start bar)
3. In the query editor that appears, type:

```
avg:system.cpu.user{*}
```

4. This shows average CPU usage across all hosts
5. Leave the visualization as **Timeseries**
6. Click outside the cell to confirm

Congratulations — you have a working Notebook!

---

## 6. Working with Cells

### Adding Cells

Click anywhere in the notebook and type **`/`** to open the command menu. You can add:

- **Timeseries** — metric queries plotted over time
- **Data Source** — logs, APM, RUM, and other data sources
- **SQL** — SQL query with tabular results
- **Table** — structured data table
- **Text** — Markdown content
- **More** — additional cell types

You can also use the **"Get started with"** quick-start bar (visible in new notebooks) to click directly into a cell type, or click **Ask Bits** to use the AI assistant.

### Reordering Cells

Hover over a cell — a **drag handle** () appears on the left. Drag cells to reorder them.

### Editing a Cell

Click on any cell to select it. For graph cells, click the **pencil icon** to edit the query.

### Deleting a Cell

Hover over a cell and click the **three-dot menu (⋯)** → **Delete**.

### Locking a Cell

Locking **freezes** a cell's data at the current moment. Useful for:
- Capturing what a graph looked like during an incident
- Creating a permanent record in a postmortem

To lock: hover over a graph cell → click the **lock icon **

> **Important:** Once locked, a cell won't update even if the time range changes. Unlock it to make it live again.

### Resizing a Cell

Drag the **bottom-right corner** of a graph cell to resize it.

---

## 7. Time Controls & Template Variables

### 7.1 Global Time Range

The time picker at the top-right applies to **all unlocked cells** in the notebook.

Common options:
- `Past 1 hour` — great for current incidents
- `Past 1 day` — daily review
- `Past 1 week` — trend analysis
- **Custom range** — set exact start and end times

> **Tip:** During an incident, set a custom range covering the incident window so everyone looking at the notebook sees the same timeframe.

### 7.2 Per-Cell Time Override

Individual graph cells can have their **own time range**, independent of the global one. Useful when you want to compare "now" vs "last week" side by side.

To set it: click the graph cell → click the **clock icon** → choose a different range.

### 7.3 Template Variables

Template variables let you **filter all cells at once** by a tag. For example:

- Set `env` = `production` → all graphs show production data
- Change to `env` = `staging` → all graphs switch to staging

**How to add a template variable:**
1. Click **+ Add Variable** in the notebook toolbar
2. Choose a tag key (e.g., `env`, `service`, `host`)
3. Set a default value
4. Now all cells that use that tag will respond to the variable

---

## 8. Collaboration Features

### 8.1 Real-Time Co-Editing

Multiple people can have the same notebook open and edit it simultaneously — just like Google Docs. Changes appear for everyone in real time.

### 8.2 Sharing

Click **Share** in the toolbar to get a shareable URL. Anyone in your Datadog organization with the right permissions can open it.

### 8.3 Comments

You can leave **comments on individual cells**:
1. Hover over a cell
2. Click the **speech bubble icon **
3. Type your comment

Use `@username` to mention a teammate — they'll get notified.

### 8.4 Saving as a Template

If you've built a notebook structure that's useful to reuse (e.g., an incident investigation template), you can save it as a **template**:
- Go to the notebook's settings menu
- Choose **Save as Template**

New notebooks can then be created from your template, pre-filled with your standard cells and structure.

### 8.5 Scheduled Snapshots

Notebooks can be configured to automatically send snapshots to email recipients on a schedule (daily, weekly, etc.). Good for recurring reports.

---

## 9. Common Use Cases

### 9.1 Incident Investigation

**Scenario:** Your alerting system fires at 2:14 AM. You open a notebook to document your investigation.

**Typical structure:**
```
## Incident: [Service] Latency Spike — [Date]

### Timeline
[Text cell with timeline of events]

### Alert Trigger
[Graph cell showing the metric that triggered the alert]

### Root Cause Investigation
[Text: hypothesis]
[Graph: relevant metric to check hypothesis]

### Resolution
[Text: what was done, by whom, when]

### Follow-up Actions
- [ ] Task 1
- [ ] Task 2
```

### 9.2 Postmortem Report

After resolving an incident, lock all the graph cells so they preserve the incident data, then write up the postmortem narrative around them.

### 9.3 Operational Runbook

```
## Runbook: Restart the Payment Service

### 1. Check Current Health
[Graph: payment service error rate — live]

### 2. Verify No Active Transactions
[Graph: active transactions count — live]

### 3. Execute Restart
Run: `kubectl rollout restart deployment/payment-service`

### 4. Confirm Recovery
[Graph: payment service error rate — live]
```

### 9.4 Performance Review

Create a notebook that shows weekly KPIs — SLOs, error rates, latency percentiles, throughput — with Markdown sections explaining each metric's context and trend.

---

## 10. Practice Exercises

> Complete these on your own to reinforce what you've learned.

### Exercise 1 — Build a System Health Notebook (15 min)

Create a notebook called `System Health Check` with:

- [ ] A text cell with a title and your name (type `/` → Text)
- [ ] A Timeseries cell showing CPU usage: `avg:system.cpu.user{*}`
- [ ] A text cell explaining what to look for (e.g., "Alert if > 80% for 5 minutes")
- [ ] A Timeseries cell showing memory usage: `avg:system.mem.used{*}`
- [ ] A Timeseries cell showing disk usage: `avg:system.disk.used{*}`
- [ ] A final text cell with a "Notes" section

### Exercise 2 — Use Template Variables (10 min)

In your notebook from Exercise 1:

- [ ] Add a template variable for `env`
- [ ] Update each graph cell query to use `$env` instead of `*`
 - Example: `avg:system.cpu.user{env:$env}`
- [ ] Switch between `production` and `staging` and observe the graphs change

### Exercise 3 — Lock and Snapshot (5 min)

- [ ] Set the time range to "Past 1 hour"
- [ ] Lock one of your graph cells using the lock icon
- [ ] Change the global time range to "Past 7 days"
- [ ] Observe: the locked cell stays fixed; unlocked cells update

### Exercise 4 — Collaboration (10 min)

- [ ] Share your notebook URL with a classmate
- [ ] Have them open it and add a comment on one of your graph cells
- [ ] Reply to their comment
- [ ] Both try editing different text cells at the same time

### Exercise 5 — Build a Mini Incident Runbook (20 min)

Create a notebook called `Runbook: High CPU Investigation` with these sections:

1. Text cell: `## Overview` — describe the scenario
2. Timeseries cell: current CPU usage across all hosts
3. Text cell: `## Step 1 — Identify the Host` — instructions
4. Timeseries cell: CPU usage broken down by host (`avg:system.cpu.user{*} by {host}`)
5. Text cell: `## Step 2 — Check Processes` — next steps
6. Text cell: `## Resolution Notes` — blank section to fill in during an incident

---

## 11. Quiz

Test your understanding. Answers are at the bottom.

**Q1.** What are the cell types available from the `/` command menu in a Datadog Notebook?

**Q2.** What does "locking" a cell do?

**Q3.** True or False: Changing the global time range also changes the time range of locked cells.

**Q4.** What is a template variable used for?

**Q5.** Name two real-world use cases for Datadog Notebooks.

**Q6.** True or False: You need to refresh the page to see another user's edits in a shared notebook.

**Q7.** Where in the Datadog UI do you find Notebooks?

**Q8.** What query would you write to see average CPU usage across all hosts?

---

### Quiz Answers

| # | Answer |
|---|---|
| Q1 | Timeseries, Text, Data Source, SQL, Table (and more via the / command menu) |
| Q2 | It freezes the cell's data at a fixed point in time — it won't update when the time range changes |
| Q3 | **False** — locked cells are not affected by the global time range |
| Q4 | To filter all cells in the notebook by a tag (e.g., environment, service, host) at once |
| Q5 | Any two of: incident investigation, postmortem reports, operational runbooks, performance reviews |
| Q6 | **False** — edits appear in real time for all viewers |
| Q7 | In the left sidebar, under the Dashboards section |
| Q8 | `avg:system.cpu.user{*}` |

---

## 12. Cheat Sheet

### Quick Actions

| Action | How |
|---|---|
| Create a notebook | Notebooks → + New Notebook |
| Add a cell | Type `/` in the notebook body → choose cell type |
| Add a Timeseries cell | Type `/` → Timeseries, or click Timeseries in quick-start bar |
| Add a Text cell | Type `/` → Text |
| Use the AI assistant | Click Ask Bits in the quick-start bar or toolbar |
| Edit a cell | Click the cell → edit inline |
| Lock a cell | Hover over cell → lock icon |
| Delete a cell | Hover → three-dot menu (⋯) → Delete |
| Share notebook | Blue Share button (top right) |
| Add a template variable | Toolbar → template variable icon |
| Clone or delete notebook | Three-dot menu (...) top right |

### Useful Metric Queries for Beginners

| What you want to see | Query |
|---|---|
| CPU usage (all hosts) | `avg:system.cpu.user{*}` |
| CPU usage (by host) | `avg:system.cpu.user{*} by {host}` |
| Memory used | `avg:system.mem.used{*}` |
| Disk usage | `avg:system.disk.used{*}` |
| Network bytes sent | `avg:system.net.bytes_sent{*}` |
| Filter to one environment | `avg:system.cpu.user{env:production}` |
| Use a template variable | `avg:system.cpu.user{env:$env}` |

### Markdown Basics for Text Cells

```markdown
# Big heading
## Medium heading
### Small heading

**bold text**
*italic text*
`code snippet`

- Bullet point
- Another point

1. Numbered step
2. Next step

> Blockquote / callout

[Link text](https://url.com)

| Column 1 | Column 2 |
|---|---|
| Cell | Cell |
```

### Tips & Tricks

- Use **Markdown cells as section headers** to structure long investigations.
- **Clone a notebook** before making big changes — it's your version control.
- **Lock cells immediately** after capturing an incident graph, before closing the tab.
- Use a **template variable for `env`** in every notebook you build — you'll thank yourself later.
- Set the **global time range first**, then add cells — it saves time tuning each graph.
- Notebooks are **not dashboards** — they're for analysis and documentation, not real-time monitoring walls.

---

## Further Reading

- [Datadog Notebooks Documentation](https://docs.datadoghq.com/notebooks/)
- [Datadog Metrics Query Language](https://docs.datadoghq.com/metrics/#querying-metrics)
- [Template Variables in Datadog](https://docs.datadoghq.com/dashboards/template_variables/)
- [Datadog SLOs Overview](https://docs.datadoghq.com/service_management/service_level_objectives/)

---

*Workshop prepared for Datadog Notebooks — Beginner track. Happy investigating! *
