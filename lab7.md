Here's a structured **1-hour hands-on workshop** built around the [Datadog Audit Trail Overview](/dash/integration/512/datadog-audit-trail-overview) dashboard:

---

# 🎓 Workshop: Datadog Audit Trail Dashboard — Clone, Customize & Investigate

**Duration:** 60 minutes | **Level:** Beginner–Intermediate
**Prerequisites:** Datadog account with Audit Trail enabled, dashboard edit permissions

---

## ⏱️ Module 1 — Orientation (10 min)

### Step 1: Explore the Out-of-the-Box Dashboard (5 min)
1. Navigate to the [Audit Trail Overview dashboard](/dash/integration/512/datadog-audit-trail-overview)
2. Observe the **13 widget groups** covering: Dashboards/Monitors/Notebooks, Log Management, Custom Metrics, APM, API Rate Limits, Sensitive Data Scanner, Security Platform, RUM & Synthetics, Access Management/SLO, Users, and API Requests
3. Use the **`$User` template variable** at the top to filter all widgets to a single user — try filtering to your own email
4. Change the **time range** to "Past 1 Day" vs "Past 7 Days" and observe how the data changes

### Step 2: Understand Audit Trail Data Sources (5 min)
1. Open [Audit Trail Explorer](/audit-trail) in a new tab
2. Note the key facets: `@evt.name` (product), `@asset.type` (resource type), `@action` (created/modified/deleted/accessed), `@usr.id` (who)
3. Click on any event to see its full JSON — this is the raw data powering the dashboard widgets

---

## ⏱️ Module 2 — Clone & Customize (15 min)

### Step 3: Clone the Dashboard (2 min)
1. On the Audit Trail dashboard, click the **⚙️ gear icon** (top right) → **Clone dashboard**
2. Rename it to: `[Workshop] My Audit Trail Dashboard`
3. You now have an editable copy — the original integration dashboard is untouched

### Step 4: Edit the Dashboard Layout (5 min)
1. Click **Edit Widgets** (pencil icon)
2. **Reorder groups**: Drag the "Users" group to the top — this is now your priority section
3. **Delete a group**: Remove the "API Requests (Across All Products)" group if not relevant to your workflow (click ✕ on the group)
4. **Collapse groups**: Note how groups help organize a large dashboard — consider which sections you'd keep vs remove for your team
5. Click **Done** to save

### Step 5: Modify an Existing Widget (8 min)
1. Enter edit mode again, find the widget **"All Audit Events by User"** in the Users group
2. Click the pencil icon on the widget to open the editor
3. **Modify the query**: Change it from all events to only `@action:modified` — this now shows only modification activity per user
4. **Change visualization**: Switch from `query_table` to `toplist` to see a ranked view
5. **Update the title** to: `Top Users by Modification Activity`
6. Click **Save**

---

## ⏱️ Module 3 — Build New Widgets (20 min)

### Step 6: Create a "Dashboard Deletion Tracker" Widget (7 min)
1. Click **+ Add Widgets**, drag a **Timeseries** widget into the "Dashboards, Monitors, and Notebooks" group
2. Set data source to **Audit Trail**
3. Query: `@evt.name:Dashboard @action:deleted`
4. Group by: `@usr.id`
5. Title: `Dashboard Deletions Over Time by User`
6. Set the display to **Bars** instead of Lines for better visibility of discrete events
7. Save the widget

### Step 7: Create a "Login Activity" Heatmap (7 min)
1. Add a **Heatmap** widget to the "Users" group
2. Data source: **Audit Trail**
3. Query: `@evt.name:Authentication @action:login`
4. X-axis: timestamp, Y-axis: `@usr.id`
5. Title: `User Login Activity Heatmap`
6. This visualizes **when** each user is active — useful for spotting unusual login patterns
7. Save the widget

### Step 8: Create a "Failed API Requests" Query Value (6 min)
1. Add a **Query Value** widget to the top of the dashboard
2. Data source: **Audit Trail**
3. Query: `@evt.name:Request @http.status_code:>=400`
4. Aggregation: **Count**
5. Set **conditional formatting**: Green < 50, Yellow 50–200, Red > 200
6. Title: `Failed API Requests (4xx/5xx)`
7. This gives a quick health indicator at a glance

---

## ⏱️ Module 4 — Template Variables & Sharing (10 min)

### Step 9: Add a New Template Variable (5 min)
1. Click the **⚙️ gear icon** → **Edit Template Variables**
2. Add a new variable:
   - **Name:** `Product`
   - **Tag/Attribute:** `@evt.name`
   - **Default:** `*`
3. Update 2–3 key widgets to include `$Product` in their query
4. Now you can filter the entire dashboard by Datadog product (APM, Log Management, Dashboard, etc.)

### Step 10: Set Up a Scheduled Report (5 min)
1. Click the **📧 Share** icon (top right) → **Schedule a Report**
2. Configure:
   - **Frequency:** Weekly, Monday at 9:00 AM
   - **Recipients:** Your team's email or Slack channel
   - **Time range:** Past 7 Days
3. This sends an automated snapshot of who changed what in the past week

---

## ⏱️ Module 5 — Wrap-up Challenge (5 min)

### 🏆 Bonus Challenge
Using what you've learned, create **one widget** that answers this question for your organization:

> *"Which users made the most changes to monitors in the past 30 days, and were any monitors deleted?"*

**Hints:**
- Query: `@evt.name:Monitor @action:(modified OR deleted)`
- Group by: `@usr.id` and `@action`
- Try a **Sunburst** or **Toplist** visualization

---

## 📋 Key Takeaways

| Concept | What You Learned |
|---------|-----------------|
| **Audit Trail Data** | Tracks who did what, when, across all Datadog products |
| **Clone → Customize** | Always clone integration dashboards before editing |
| **Template Variables** | Enable dynamic filtering across all widgets with `$variable` |
| **Widget Types** | Different viz types serve different questions (tables for detail, timeseries for trends, heatmaps for patterns) |
| **Key Facets** | `@evt.name`, `@action`, `@asset.type`, `@usr.id` are the core dimensions |

---

Want me to save this workshop as a **Datadog Notebook** so participants can follow along interactively?
