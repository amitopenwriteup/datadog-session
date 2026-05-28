#  Datadog Sheets Workshop
### Hands-On Guide to Data Analysis, Live Queries & Spreadsheet Workflows

---

## Table of Contents

1. [Introduction to Datadog Sheets](#1-introduction-to-datadog-sheets)
2. [Getting Started — Interface Overview](#2-getting-started--interface-overview)
3. [Connecting Your Data Sources](#3-connecting-your-data-sources)
4. [Working with Tables](#4-working-with-tables)
5. [Data Sources Deep Dive](#5-data-sources-deep-dive)
6. [Calculated Columns & Formulas](#6-calculated-columns--formulas)
7. [Lookup & Joins](#7-lookup--joins)
8. [Pivot Tables](#8-pivot-tables)
9. [Graphs & Visualizations](#9-graphs--visualizations)
10. [Templates & Use Cases](#10-templates--use-cases)
11. [Best Practices](#11-best-practices)
12. [Hands-On Labs](#12-hands-on-labs)

---

## 1. Introduction to Datadog Sheets

**Datadog Sheets** is a spreadsheet-style interface built directly into Datadog that lets you query, analyze, and manipulate live observability data — without exporting to external tools.

### Why Use Sheets?

| Traditional Workflow | With Datadog Sheets |
|----------------------|---------------------|
| Export logs to CSV → Open in Excel | Query logs live in Sheets |
| Write SQL on metrics warehouse | Point-and-click pivot tables |
| Manual cost breakdowns in Sheets | Auto-populated AWS/GCP cost tables |
| Switch between tools to correlate | All data in one place |

### Key Capabilities

- **Live data** — Sheets queries Datadog in real time; no stale exports
- **Calculated columns** — Add custom formulas on top of raw data
- **Pivot tables** — Aggregate and slice data without code
- **Lookups** — Join data across multiple sources
- **Graphs** — Visualize directly from the spreadsheet
- **Templates** — Pre-built analyses for common use cases

---

## 2. Getting Started — Interface Overview

### Navigation

```
Datadog Sidebar → Notebooks / Sheets icon
                  (or search "Sheets" in Ctrl+K)
```

### Main Layout

```
┌─────────────────────────────────────────────────────────┐
│  Getting Started Panel (left)  │  Canvas / Sheets (right)│
│                                 │                         │
│  ┌─ Connect your data ───────┐  │  Get Started with Sheets│
│  │  + Add Table              │  │                         │
│  └───────────────────────────┘  │  ┌── Connect to Data ──┐│
│                                 │  │ Logs  RUM  Ref Tables││
│  ┌─ Choose a template ───────┐  │  │ Metrics LLM  Events ││
│  │  Cloud Security Identity  │  │  └─────────────────────┘│
│  │  AWS cost analysis        │  │                         │
│  │  Open AI cost analysis    │  │  Recommended Templates  │
│  └───────────────────────────┘  └─────────────────────────┘
```

### Toolbar Icons (Left Sidebar)

| Icon | Function |
|------|----------|
| 🗃️ | Add Table (data source) |
| Σ | Formulas / Calculated Columns |
| 📊 | Charts & Graphs |
| 🧮 | Pivot Table |
| 🗂️ | Reference Table view |

---

## 3. Connecting Your Data Sources

### Step 1 — Add a Table

Click **"Add Table"** in the Getting Started panel or use the sidebar icon.

### Available Data Sources

#### Logs
```
Source:   Logs
Query:    source:nginx status:error
Time:     Past 1 Hour / Past 24 Hours / Custom
Columns:  @http.status_code, @http.url, @duration, host
Limit:    Up to 10,000 rows
```

#### Metrics
```
Source:   Metrics
Metric:   system.cpu.user
Filter:   env:prod
Group by: host, service
Aggregation: avg / sum / max / min / count
```

#### RUM (Real User Monitoring)
```
Source:   RUM
Events:   page views / actions / errors / resources
Filter:   @application.name:my-app
Columns:  @view.url, @session.id, @duration, country
```

#### Cloud Cost
```
Source:   Cloud Cost
Provider: AWS / GCP / Azure
Filter:   team:platform, env:prod
Columns:  service_name, usage_type, cost, date
Time:     Last 3 months
```

#### Reference Tables
```
Source:   Reference Table
Use for:  Enrichment lookups (e.g., team ownership maps,
          host → service mappings, cost center codes)
```

#### Events
```
Source:   Events
Query:    source:jenkins status:error
Columns:  title, tags, alert_type, timestamp
```

#### LLM Observability
```
Source:   LLM Observability
Data:     Token usage, latency, error rates per model/app
Use for:  AI cost analysis and quality tracking
```

---

## 4. Working with Tables

### Table Anatomy

```
┌──────────┬─────────────┬──────────┬────────────┬─────────────┐
│ Row # │ host        │ service  │ cpu_usage  │ timestamp   │
├──────────┼─────────────┼──────────┼────────────┼─────────────┤
│ 1        │ web-01      │ api      │ 87.4       │ 2024-03-15  │
│ 2        │ web-02      │ api      │ 62.1       │ 2024-03-15  │
│ 3        │ db-01       │ postgres │ 45.8       │ 2024-03-15  │
└──────────┴─────────────┴──────────┴────────────┴─────────────┘
```

### Column Operations

**Right-click any column header to:**
- Rename column
- Sort ascending / descending
- Filter values
- Hide column
- Add calculated column based on this column
- Delete column

### Filtering Data

Click the **filter icon** on any column:

```
Text columns:    contains / equals / starts with
Numeric columns: > / < / = / between
Date columns:    before / after / within range
Tag columns:     in list / not in list
```

### Sorting

- Click column header once → Sort ascending
- Click again → Sort descending
- Hold Shift + click → Multi-column sort

### Time Range

The time picker (top right) controls the query window:
```
Past 1 Hour   (default)
Past 4 Hours
Past 1 Day
Past 7 Days
Past 30 Days
Custom Range  → pick start/end timestamps
```

---

## 5. Data Sources Deep Dive

### 5.1 Logs Analysis

**Use case:** Find the top error-producing endpoints in the past hour.

```
Table config:
  Source: Logs
  Query:  status:error
  Group:  @http.url
  Columns: @http.url, count(*), @duration_p99

Result:
  /api/checkout     → 1,247 errors, p99: 2,340ms
  /api/user/login   →   342 errors, p99:   890ms
  /api/search       →    89 errors, p99:   420ms
```

### 5.2 AWS Cost by Team

**Use case:** Break down AWS spend per team for the past 3 months.

```
Table config:
  Source: Cloud Cost
  Filter: provider:aws
  Group:  team, aws_service
  Columns: team, aws_service, sum(cost), month

Template available: "AWS cost analysis by team past 3 months"
```

### 5.3 Open AI Cost Analysis

**Use case:** Track AI/LLM spending by model and application.

```
Table config:
  Source: LLM Observability
  Columns: model_name, app_name, total_tokens,
           estimated_cost, avg_latency_ms

Template available: "Open AI cost analysis past 3 months"
```

### 5.4 Cloud Security Identity Risks

**Use case:** Analyze IAM/identity risk findings.

```
Table config:
  Source: Cloud Security
  Columns: resource_name, finding_type, severity,
           status, cloud_account, last_seen

Template available: "Cloud Security Identity Risks"
Severity filter: critical, high, medium, low
```

### 5.5 RUM Performance Analysis

**Use case:** Identify slow pages affecting real users.

```
Table config:
  Source: RUM
  Query:  @view.loading_time:>3000
  Columns: @view.url, @view.loading_time, 
           @browser.name, country, count(session)
```

---

## 6. Calculated Columns & Formulas

### Adding a Calculated Column

1. Click the **Σ (sigma)** icon in the sidebar
2. Or right-click a column → **"Add calculated column"**
3. Enter a formula name and expression

### Formula Syntax

#### Basic Arithmetic

```
# Error rate percentage
error_rate = (error_count / total_count) * 100

# Cost per request
cost_per_req = total_cost / request_count

# Latency in seconds
latency_sec = latency_ms / 1000
```

#### Conditional Logic

```
# Severity label
severity_label = IF(error_rate > 10, "Critical",
                 IF(error_rate > 5,  "High",
                 IF(error_rate > 1,  "Medium", "Low")))

# Over budget flag
over_budget = IF(monthly_cost > budget_limit, "YES", "NO")
```

#### Text Operations

```
# Combine host and environment
display_name = CONCAT(host, " (", env, ")")

# Extract service from tag string
service = SPLIT(tags, "service:", 1)

# Uppercase environment
env_label = UPPER(environment)
```

#### Numeric Functions

```
ROUND(value, 2)        ← Round to 2 decimal places
ABS(value)             ← Absolute value
MIN(col_a, col_b)      ← Minimum of two columns
MAX(col_a, col_b)      ← Maximum of two columns
SUM(col_a)             ← Sum of a column (in pivot context)
AVG(col_a)             ← Average
```

#### Date Functions

```
DATE_DIFF(end_time, start_time, "hours")   ← Time difference
DATE_FORMAT(timestamp, "YYYY-MM-DD")       ← Format date
DATE_TRUNC(timestamp, "day")               ← Truncate to day
NOW()                                      ← Current timestamp
```

---

## 7. Lookup & Joins

### What is a Lookup?

Lookups let you **enrich one table with data from another** — similar to VLOOKUP in Excel or a JOIN in SQL.

### Example: Enrich Cost Data with Team Names

```
Table A: AWS Costs
  account_id | service | cost

Table B: Reference Table (account → team mapping)
  account_id | team_name | cost_center

Lookup config:
  Join on:  Table_A.account_id = Table_B.account_id
  Add from B: team_name, cost_center

Result:
  account_id | service | cost  | team_name | cost_center
  123456     | EC2     | $450  | Platform  | CC-001
```

### Setting Up a Lookup

1. Open your primary table
2. Click **"Add Lookup Column"** (from column right-click or toolbar)
3. Select the reference table or second data source
4. Choose the **join key** (matching column in both tables)
5. Select which columns to pull in
6. Name the new lookup column

### Lookup Types

| Type | Behavior |
|------|----------|
| **Left Join** | Keep all rows from primary table; NULL if no match |
| **Inner Join** | Only keep rows that match in both tables |
| **First match** | If multiple matches, take the first result |

---

## 8. Pivot Tables

### Creating a Pivot Table

1. Add a table with your data source
2. Click **🧮 Pivot** icon in the toolbar
3. Configure dimensions and measures

### Pivot Configuration

```
Rows:    team, service          ← What to group by
Columns: month                  ← Optional column grouping
Values:  SUM(cost)              ← What to aggregate
         COUNT(errors)
         AVG(latency_ms)
         MAX(cpu_usage)
```

### Example: AWS Cost by Team × Service

```
Pivot Output:

              │  EC2    │  RDS    │  S3     │  Lambda │  TOTAL
──────────────┼─────────┼─────────┼─────────┼─────────┼────────
Platform      │ $4,200  │ $1,800  │  $320   │  $150   │ $6,470
Backend       │ $2,100  │ $3,400  │  $180   │  $890   │ $6,570
Frontend      │   $800  │     -   │  $420   │  $200   │ $1,420
Data          │ $1,500  │ $5,200  │  $950   │  $340   │ $7,990
──────────────┼─────────┼─────────┼─────────┼─────────┼────────
TOTAL         │ $8,600  │$10,400  │$1,870   │$1,580   │$22,450
```

### Sorting Pivot Results

- Click any column header to sort
- Use the **"Sort by"** dropdown for total-based sorting
- Toggle **"Show totals"** for row and column sums

---

## 9. Graphs & Visualizations

### Adding a Graph

1. Click the **📊 Chart** icon in the toolbar
2. Select your data table as the source
3. Choose chart type and configure axes

### Chart Types

| Chart Type | Best For |
|------------|----------|
| **Bar Chart** | Comparing values across categories |
| **Line Chart** | Trends over time |
| **Pie / Donut** | Proportional breakdown |
| **Scatter Plot** | Correlation between two metrics |
| **Heatmap** | Density and distribution patterns |
| **Area Chart** | Cumulative trends over time |

### Bar Chart: Error Count by Service

```
Chart type: Bar
X-axis:     service_name
Y-axis:     SUM(error_count)
Color by:   environment
Sort:       Descending by Y value
Title:      Top Error-Producing Services
```

### Line Chart: Cost Trend Over Time

```
Chart type: Line
X-axis:     date (day)
Y-axis:     SUM(daily_cost)
Color by:   team
Time range: Last 90 days
Title:      AWS Cost Trend by Team
```

### Pie Chart: Cost Distribution by Service

```
Chart type: Donut
Segments:   aws_service
Values:     SUM(cost)
Show:        % labels, legend
Title:      AWS Spend Distribution
```

---

## 10. Templates & Use Cases

### Built-in Templates (visible on your screen)

#### 1. Cloud Security Identity Risks
```
Purpose:  Audit IAM/identity risk posture
Data:     Cloud Security findings
Columns:  Resource, finding type, severity, status
Analysis: Filter by critical/high severity; track open vs resolved
```

#### 2. AWS Cost Analysis by Team — Past 3 Months
```
Purpose:  FinOps — track cloud spend per team
Data:     Cloud Cost (AWS)
Columns:  Team, service, month, cost
Analysis: Pivot by team × service; spot budget overruns
```

#### 3. Open AI Cost Analysis — Past 3 Months
```
Purpose:  Track AI/LLM API spend
Data:     LLM Observability
Columns:  Model, app, tokens, estimated cost, latency
Analysis: Cost per model, token efficiency ratio
```

### Common Custom Analyses

#### SRE: Incident Impact Analysis
```
Source 1: Logs (errors during incident window)
Source 2: RUM (user sessions during same window)
Join on:  timestamp range
Output:   Errors → User impact correlation
```

#### FinOps: Cost per Request
```
Source 1: Cloud Cost (AWS costs per service)
Source 2: Metrics (request counts per service)
Calculated: cost_per_1000_requests = (cost / requests) * 1000
```

#### Security: Unresolved High-Severity Findings
```
Source:  Cloud Security
Filter:  severity:high OR severity:critical
         status:open
Sort by: last_seen DESC
Output:  Prioritized remediation list
```

---

## 11. Best Practices

### Performance

```
✅ Filter data at source level — add query filters before pulling data
✅ Limit time ranges — avoid "All Time" queries on large datasets
✅ Use group-by in source queries to pre-aggregate
✅ Limit columns to what you need — don't pull 50 columns if you need 8
✅ Use Reference Tables for static enrichment data (not live queries)
```

### Data Quality

```
✅ Always validate row counts after connecting a data source
✅ Check for NULL values before dividing columns
✅ Use IF() to handle edge cases in calculated columns:
   safe_rate = IF(total = 0, 0, errors / total * 100)
✅ Document calculated column formulas with descriptive names
```

### Collaboration

```
✅ Name sheets descriptively: "AWS Cost Q1 2024 - Platform Team"
✅ Add a text block above each table explaining its purpose
✅ Share sheets via URL — collaborators see live data
✅ Use templates for recurring analysis (run monthly/weekly)
✅ Pin important sheets to your team's dashboard
```

### Security

```
✅ Sheets respect Datadog RBAC — users only see data they have access to
✅ Reference Tables with sensitive data (PII mappings) → restrict permissions
✅ Don't export sensitive cost data to CSV unnecessarily — share the Sheet URL
```

---

## 12. Hands-On Labs

### Lab 1 — Connect Log Data & Find Top Errors (15 min)

**Objective:** Build a table of the top 10 error-producing services.

```
Steps:
1. Open Datadog Sheets (Sidebar → Sheets icon)
2. Click "Add Table"
3. Select source: Logs
4. Set query: status:error
5. Time range: Past 1 Hour
6. Add columns: service, @http.status_code, @http.url, @duration
7. Add a filter: @http.status_code >= 500
8. Sort by: count DESC
9. Add calculated column:
   Name: error_pct
   Formula: (error_count / total_count) * 100
10. Observe the top services by error volume
```

**Expected output:** A ranked table of services with error counts and percentages.

---

### Lab 2 — AWS Cost Breakdown Using Template (10 min)

**Objective:** Use the built-in AWS cost template to analyze team spend.

```
Steps:
1. In Sheets, click "Choose a template"
2. Select: "AWS cost analysis by team past 3 months"
3. Let the template load with your data
4. Add a Pivot Table:
   - Rows: team
   - Columns: month
   - Values: SUM(cost)
5. Sort by total cost descending
6. Add a Bar Chart:
   - X: team
   - Y: SUM(cost)
   - Color by: month
7. Identify the highest-spending team
```

**Expected output:** A pivot table and bar chart of AWS costs per team per month.

---

### Lab 3 — Create a Calculated Column for Cost Per Service (15 min)

**Objective:** Calculate cost efficiency metrics.

```
Steps:
1. Start from the AWS cost table (Lab 2 or new connection)
2. Add a Metrics table:
   - Metric: trace.web.request.hits
   - Group by: service
   - Time: Last 30 days
3. Perform a Lookup:
   - Join: cost_table.service = metrics_table.service
   - Pull in: request_count
4. Add Calculated Column:
   Name: cost_per_1k_requests
   Formula: ROUND((monthly_cost / request_count) * 1000, 4)
5. Sort by cost_per_1k_requests DESC
6. Flag outliers:
   Name: efficiency_flag
   Formula: IF(cost_per_1k_requests > 0.50, "Review", "OK")
```

**Expected output:** Cost efficiency scores per service with outlier flagging.

---

### Lab 4 — Security Findings Analysis (10 min)

**Objective:** Use the Cloud Security template to triage identity risks.

```
Steps:
1. Click "Choose a template"
2. Select: "Cloud Security Identity Risks"
3. Filter to: severity = "critical" OR severity = "high"
4. Filter to: status = "open"
5. Add Calculated Column:
   Name: days_open
   Formula: DATE_DIFF(NOW(), first_seen, "days")
6. Sort by: days_open DESC
7. Add a Pie Chart:
   - Segments: finding_type
   - Values: COUNT(resource)
   - Title: Open High-Severity Findings by Type
```

**Expected output:** A prioritized list of security findings with age, plus a breakdown chart.

---

### Lab 5 — RUM Performance Deep Dive (20 min)

**Objective:** Identify slow pages impacting real users.

```
Steps:
1. Add Table → Source: RUM
2. Columns: @view.url, @view.loading_time, @session.country,
            @browser.name, @user.id
3. Filter: @view.loading_time > 3000 (over 3 seconds)
4. Time range: Past 24 Hours
5. Create Pivot Table:
   - Rows: @view.url
   - Values: AVG(@view.loading_time), COUNT(@session.id)
6. Add Calculated Column:
   Name: p90_impact_score
   Formula: avg_loading_time * session_count / 1000
7. Sort by p90_impact_score DESC
8. Create Line Chart:
   - X: hour (time bucket)
   - Y: AVG(loading_time)
   - Color by: @view.url (top 5)
   - Title: Page Load Time Trend (Slow Pages)
```

**Expected output:** Ranked table of slowest pages by user impact, with trend chart.

---

## Appendix

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Global Datadog search |
| `Click column header` | Sort ascending/descending |
| `Right-click column` | Column options menu |
| `Ctrl+Z` | Undo last action |
| `Ctrl+F` | Find in table |

### Formula Reference Card

```
# Math
+  -  *  /  ^  MOD(a, b)

# Comparison
>  <  >=  <=  =  !=

# Logic
AND(a, b)    OR(a, b)    NOT(a)
IF(condition, true_val, false_val)

# Text
CONCAT(a, b, ...)    UPPER(s)    LOWER(s)
SPLIT(s, delimiter, index)
TRIM(s)              LEN(s)
REPLACE(s, old, new)

# Numeric
ROUND(n, decimals)   FLOOR(n)    CEILING(n)
ABS(n)               MIN(a, b)   MAX(a, b)
SUM(col)             AVG(col)    COUNT(col)

# Date/Time
NOW()
DATE_DIFF(end, start, "days"/"hours"/"minutes")
DATE_FORMAT(ts, "YYYY-MM-DD HH:mm")
DATE_TRUNC(ts, "day"/"week"/"month")
```

### Data Source Quick Reference

| Source | Best For | Max Rows |
|--------|----------|----------|
| Logs | Error analysis, audit logs | 10,000 |
| Metrics | Time-series performance data | Aggregated |
| RUM | User experience, session data | 10,000 |
| Cloud Cost | FinOps, budget analysis | Aggregated |
| Events | Deployments, incidents | 10,000 |
| LLM Observability | AI/ML cost & quality | Aggregated |
| Reference Tables | Static enrichment data | 5,000 |

---

### Workshop Resources

- 📘 Sheets Docs: https://docs.datadoghq.com/sheets/
- 🎓 Learning Center: https://learn.datadoghq.com
- 💰 Cloud Cost Docs: https://docs.datadoghq.com/cloud_cost_management/
- 🔒 Cloud Security Docs: https://docs.datadoghq.com/security/cloud_security_management/
- 🤖 LLM Observability Docs: https://docs.datadoghq.com/llm_observability/

---

*Workshop Version 1.0 | Datadog Sheets Training Guide*
