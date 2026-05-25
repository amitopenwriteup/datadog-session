

---

# 🎓 Workshop: Clone & Customize Host Metrics into a P1 Incident Dashboard

**Duration:** 60 minutes | **Level:** Beginner–Intermediate
**Source Dashboard:** [Host Metrics](/dash/integration/1808/host-metrics)
**Goal:** Transform the generic Host Metrics dashboard into a focused **P1 incident response dashboard** with critical thresholds, alerts-at-a-glance, and faster triage.

---

## ⏱️ Module 1 — Orientation (10 min)

### Step 1: Explore the Source Dashboard (5 min)

1. Open the [Host Metrics dashboard](/dash/integration/1808/host-metrics)
2. Review the **6 widget groups** and what they track:

| Group | Key Metrics | P1 Relevance |
|-------|-------------|--------------|
| **Overview** | CPU %, Memory %, Uptime, NTP offset | 🔴 Critical — first thing to check |
| **CPU** | CPU usage breakdown, iowait, load averages | 🔴 High CPU = app degradation |
| **Memory** | RAM total/used, swap, cached | 🔴 OOM risk |
| **Disk** | Disk usage %, latency, I/O wait | 🟡 Disk full = cascading failures |
| **Network** | Bytes sent/received, retransmits | 🟡 Network issues = timeouts |
| **Start Here** | Info notes | ⚪ Not needed in P1 dashboard |

3. Use the **`$host` template variable** — filter to a specific host and see how all widgets respond

### Step 2: Identify P1 Thresholds (5 min)

Discuss with your team and note down your org's critical thresholds:

| Metric | 🟢 Healthy | 🟡 Warning | 🔴 Critical (P1) |
|--------|-----------|------------|-------------------|
| CPU Usage | < 70% | 70–85% | > 85% |
| Memory Usage | < 75% | 75–90% | > 90% |
| Disk Usage | < 70% | 70–85% | > 85% |
| Load Average (norm) | < 1.0 | 1.0–2.0 | > 2.0 |
| Disk I/O Latency | < 10ms | 10–50ms | > 50ms |
| CPU iowait | < 10% | 10–25% | > 25% |

> 💡 **Adjust these to your environment.** These are starting points — your team knows your infra best.

---

## ⏱️ Module 2 — Clone & Clean Up (10 min)

### Step 3: Clone the Dashboard (2 min)

1. On the [Host Metrics dashboard](/dash/integration/1808/host-metrics), click the **⚙️ gear icon** (top-right)
2. Select **Clone dashboard**
3. Rename to: **`[P1] Host Health — Incident Response`**
4. Click **Confirm**

### Step 4: Remove Non-Essential Content (3 min)

1. Click **Edit Widgets** (pencil icon)
2. **Delete the "Start Here" group** — it's just onboarding text, not useful during a P1
3. Click **Done** to save

### Step 5: Reorder Groups for P1 Triage Flow (5 min)

During a P1, you scan top-to-bottom. Reorder groups by incident priority:

1. Enter edit mode
2. Drag groups into this order:
   - **① Overview** — instant health snapshot
   - **② CPU** — most common P1 cause
   - **③ Memory** — OOM / swap thrashing
   - **④ Disk** — full disk / high latency
   - **⑤ Network** — connectivity issues
3. Click **Done**

---

## ⏱️ Module 3 — Add P1 Conditional Formatting (15 min)

### Step 6: Add Thresholds to "CPU Usage (%)" Query Value (5 min)

1. Edit the **"CPU Usage (%)"** widget in the Overview group (pencil icon)
2. Go to the **Conditional Formatting** section
3. Add rules:
   - 🟢 Green: value **< 70**
   - 🟡 Yellow: value **>= 70**
   - 🔴 Red: value **>= 85**
4. The number now changes color based on severity — instant visual during a P1
5. **Save**

### Step 7: Add Thresholds to "Memory Usage (%)" Query Value (5 min)

1. Edit the **"Memory Usage (%)"** widget
2. Add conditional formatting:
   - 🟢 Green: value **> 25** (remember, this metric is `pct_usable`, so higher = more free)
   - 🟡 Yellow: value **<= 25**
   - 🔴 Red: value **<= 10**
3. **Save**

### Step 8: Add Thresholds to "% CPU iowait" Query Value (5 min)

1. Edit the **"% CPU iowait"** widget in the CPU group
2. Add conditional formatting:
   - 🟢 Green: value **< 10**
   - 🟡 Yellow: value **>= 10**
   - 🔴 Red: value **>= 25**
3. **Save**

> ✅ **Checkpoint:** Your Overview and CPU groups now have color-coded health indicators that turn red during a P1.

---

## ⏱️ Module 4 — Build P1-Specific Widgets (15 min)

### Step 9: Create a "Top Hosts by CPU" Top List (5 min)

1. Click **+ Add Widgets**, drag a **Top List** into the CPU group
2. Configure:
   - **Metric:** `system.cpu.user`
   - **Aggregation:** `avg` by `host`
   - **Title:** `Top Hosts by CPU Usage`
3. Add conditional formatting:
   - 🟢 Green: < 70
   - 🟡 Yellow: >= 70
   - 🔴 Red: >= 85
4. **Save**

**Why:** During a P1, you need to immediately identify *which* host is the problem — not just see an average.

### Step 10: Create a "Disk Space Critical" Top List (5 min)

1. Drag a **Top List** into the Disk group
2. Configure:
   - **Metric:** `system.disk.in_use`
   - **Aggregation:** `max` by `host`, `device`
   - **Filter:** `!device:/dev/loop*` (exclude loop devices)
   - **Title:** `Disk Usage by Host & Device (%)`
3. Add conditional formatting:
   - 🟢 Green: < 0.70
   - 🟡 Yellow: >= 0.70
   - 🔴 Red: >= 0.85
4. **Save**

> ⚠️ Note: `system.disk.in_use` is a ratio (0–1), not a percentage — so 0.85 = 85%.

### Step 11: Create a "Load Average Spike" Timeseries with Threshold Marker (5 min)

1. Drag a **Timeseries** into the CPU group
2. Configure:
   - **Queries:**
     - `avg:system.load.norm.1{$host}` (label: "1 min")
     - `avg:system.load.norm.15{$host}` (label: "15 min")
   - **Add a Marker:** Horizontal line at **y = 2.0**, color **Red**, label: `P1 THRESHOLD`
   - **Title:** `Normalized Load Average with P1 Threshold`
3. **Save**

**Why:** The red threshold line makes it instantly clear when load crosses into P1 territory.

---

## ⏱️ Module 5 — Template Variables & Final Touches (7 min)

### Step 12: Add an Environment Template Variable (3 min)

1. Click **⚙️ gear icon** → **Edit Template Variables**
2. Add a new variable:
   - **Name:** `env`
   - **Tag/Attribute:** `env`
   - **Default:** `production`
3. **Save**
4. Update key widget queries to include `$env` alongside `$host`:
   - Example: `avg:system.cpu.user{$host,$env}` → now you can scope the whole dashboard to `env:production` during a P1

### Step 13: Add a P1 Runbook Note (2 min)

1. Drag a **Notes & Links** widget to the very top of the dashboard
2. Add content:
   ```
   ## 🚨 P1 Incident Runbook
   1. Check Overview section — any red indicators?
   2. Identify the affected host using Top Lists
   3. Check CPU → Memory → Disk → Network (in order)
   4. Escalation: #incident-channel | On-call: [PagerDuty link]
   ```
3. Set background color to **Yellow** for visibility
4. **Save**

### Step 14: Set Default Timeframe (2 min)

1. Set the dashboard time range to **Past 1 Hour** — this is the most useful window during active incidents
2. Click the **clock icon** → select **1 Hour**
3. This becomes the default view when anyone opens the dashboard during a P1

---

## ⏱️ Wrap-up & Challenge (3 min)

### ✅ What You Built

| What Changed | Before (Host Metrics) | After (P1 Dashboard) |
|-------------|----------------------|---------------------|
| **Groups** | 6 groups with "Start Here" | 5 groups, ordered by triage priority |
| **Query Values** | Plain numbers | Color-coded with P1 thresholds |
| **Top Lists** | None | 2 new — identify problem hosts instantly |
| **Threshold Markers** | None | Load average P1 line |
| **Template Vars** | `$host` only | `$host` + `$env` |
| **Runbook** | None | Embedded at top |

### 🏆 Bonus Challenge

Create a **Query Value** widget showing **Network Retransmit Rate** with conditional formatting:
- **Metric:** `system.net.tcp.retrans_segs` with `.as_rate()`
- 🟢 Green: < 100/s | 🟡 Yellow: >= 100/s | 🔴 Red: >= 500/s

---

### 📚 Key Takeaways

| Concept | What You Learned |
|---------|-----------------|
| **Clone → Customize** | Never edit integration dashboards directly |
| **Conditional Formatting** | Color-coded thresholds turn dashboards into instant health indicators |
| **Top Lists** | Essential during P1 to pinpoint *which* host is the problem |
| **Threshold Markers** | Red lines on timeseries make anomalies obvious |
| **Triage Order** | Group ordering matters — CPU → Memory → Disk → Network |
| **Embedded Runbook** | A note widget at the top guides responders during high-stress P1s |
