# Datadog Lab 1 — Verifying Agent Health & APM Ingestion

**Stack:** Ubuntu Server · Nginx · Datadog Agent with APM  
**Duration:** 1.5 Hours  
**Goal:** Confirm the Datadog Agent is healthy, APM is receiving traces from your Nginx app, and understand the Datadog UI layout before deeper investigation.

---

## Pre-Lab Setup — Nginx Installation on Ubuntu

> Complete this section before starting any Datadog UI steps. All commands run as a non-root user with `sudo` access.

### Step 1 — Update the system

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2 — Install Nginx

```bash
sudo apt install nginx -y
```

Verify the installation:

```bash
nginx -v
# Expected output: nginx version: nginx/1.x.x
```

### Step 3 — Start and enable Nginx

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Confirm Nginx is active:

```bash
sudo systemctl status nginx
# Look for: Active: active (running)
```

### Step 4 — Allow Nginx through the firewall

```bash
sudo ufw allow 'Nginx Full'
sudo ufw status
```

### Step 5 — Verify Nginx is serving traffic

```bash
curl -I http://localhost
# Expected: HTTP/1.1 200 OK
```

You can also open `http://<your-server-ip>` in a browser — you should see the default Nginx welcome page.

### Step 6 — Configure Nginx for Datadog APM tracing

Edit the main Nginx config to add an APM-compatible log format:

```bash
sudo tee -a /etc/nginx/nginx.conf > /dev/null <<'EOF'

http {
    log_format datadog '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $body_bytes_sent '
                       '"$http_referer" "$http_user_agent" '
                       'rt=$request_time';

    access_log /var/log/nginx/access.log datadog;
}
EOF
```

Save and test the config:

```bash
sudo nginx -t
# Expected: syntax is ok / test is successful
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

### Step 7 — Install the Datadog Agent

Replace `<YOUR_DD_API_KEY>` with your actual key from the Datadog UI under **Organization Settings → API Keys**:

```bash
DD_API_KEY=<YOUR_DD_API_KEY> \
DD_SITE="datadoghq.com" \
bash -c "$(curl -L https://s3.amazonaws.com/dd-agent-datadoghq-com/scripts/install_script_agent7.sh)"
```

### Step 8 — Enable APM in the Datadog Agent config

```bash
sudo nano /etc/datadog-agent/datadog.yaml
```

Ensure these lines are present and uncommented:

```yaml
apm_config:
  enabled: true

env: prod
tags:
  - env:prod
  - service:nginx
```

Restart the Agent to apply changes:

```bash
sudo systemctl restart datadog-agent
```

Verify the Agent is running and APM is active:

```bash
sudo datadog-agent status | grep -A 10 "APM Agent"
# Expected: Running: true
```

### Step 9 — Configure the Nginx integration for the Datadog Agent

Create the Nginx integration config:

```bash
sudo nano /etc/datadog-agent/conf.d/nginx.d/conf.yaml
```

Add the following:

```yaml
init_config:

instances:
  - nginx_status_url: http://localhost/nginx_status
```

Enable the Nginx status endpoint in your server block:

```bash
sudo nano /etc/nginx/sites-available/default
```

Add inside the `server {}` block:

```nginx
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}
```

Reload Nginx and restart the Agent:

```bash
sudo nginx -t && sudo systemctl reload nginx
sudo systemctl restart datadog-agent
```

### ✅ Pre-Lab Checkpoint

Run all four checks — each should return clean output before proceeding:

```bash
sudo systemctl status nginx           # Active: active (running)
sudo systemctl status datadog-agent   # Active: active (running)
sudo datadog-agent status | grep -A 5 "APM Agent"   # Running: true
curl -I http://localhost              # HTTP/1.1 200 OK
```

---

## Generating Workload on Nginx

> Run this on your Ubuntu server to produce real traffic in Datadog. Without traffic, APM traces and metrics will be empty in the UI.

### Option A — Quick manual burst (1 min)

Send 200 requests using `curl` in a loop:

```bash
for i in $(seq 1 200); do
  curl -s http://localhost > /dev/null
  sleep 0.1
done
```

### Option B — Sustained load with `ab` (Apache Benchmark)

Install and run:

```bash
sudo apt install apache2-utils -y

# 500 total requests, 10 concurrent users
ab -n 500 -c 10 http://localhost/

# Run continuously for 10 minutes in the background
ab -n 5000 -c 10 -t 600 http://localhost/ &
```

### Option C — Realistic mixed traffic with `wrk`

Install `wrk`:

```bash
sudo apt install wrk -y
```

Run a 2-minute load test with 4 threads and 20 connections:

```bash
wrk -t4 -c20 -d120s http://localhost/
```

Generate traffic across multiple paths to create varied resource names in APM:

```bash
# Homepage traffic
wrk -t2 -c10 -d120s http://localhost/ &

# Simulate 404s to generate error traces
for i in $(seq 1 50); do
  curl -s http://localhost/nonexistent > /dev/null
  sleep 1
done &

wait
```

### Option D — Continuous background load (recommended for full lab duration)

Create a reusable load script:

```bash
cat << 'EOF' > ~/load_nginx.sh
#!/bin/bash
echo "Starting continuous Nginx load — Ctrl+C to stop"
while true; do
  curl -s http://localhost/ > /dev/null
  curl -s http://localhost/index.html > /dev/null
  curl -s http://localhost/nonexistent > /dev/null   # generates 404 error traces
  sleep 0.5
done
EOF

chmod +x ~/load_nginx.sh
```

Run it in the background before starting the Datadog UI steps:

```bash
nohup ~/load_nginx.sh &
echo "Load script PID: $!"
```

Stop it after the lab:

```bash
pkill -f load_nginx.sh
```

### Verify traffic is flowing

```bash
# Confirm requests are hitting Nginx
sudo tail -f /var/log/nginx/access.log

# Confirm traces are being sent to Datadog
sudo datadog-agent status | grep -A 15 "APM Agent"
# Look for: Traces received, Traces sent
```

> Allow **2–3 minutes** after starting the load before checking the Datadog UI — traces need time to be ingested and indexed.

---

## Part 1 — Agent Status in the Datadog UI (20 min)

**Navigation:** `Infrastructure → Infrastructure List`

### Steps

1. Log in to [https://app.datadoghq.com](https://app.datadoghq.com)
2. In the left sidebar, go to **Infrastructure → Infrastructure List**
3. Locate your Ubuntu server hostname in the list
   - 🟢 **Green dot** = Agent is reporting
   - 🔴 **Grey or red dot** = Agent is not sending data — check connectivity before proceeding
4. Click the hostname to open the **Host Detail Panel** on the right
5. In the Host Detail Panel, review:
   - **Apps running** — confirm `nginx` appears in the process list
   - **Agent version** — note the version shown; APM requires Agent 6+
   - **Tags** — check that environment tags (e.g. `env:prod`) are present
6. Click **View Host Dashboard** (button at top of the Host Detail Panel)
7. On the Host Dashboard, scroll through the following widgets and note baseline values:
   - CPU Utilization
   - Memory Used
   - I/O Wait
   - Network bytes sent/received

### ✅ Checkpoint

> 📸 Screenshot the Host Dashboard. You will compare this baseline in Labs 4 and 5.

---

## Part 2 — Navigating to APM Home (20 min)

**Navigation:** `APM → Services`

### Steps

1. In the left sidebar, go to **APM → Services**
2. You will land on the **Service Catalog**
3. Locate your Nginx service (it may appear as `nginx` or the name set in `DD_SERVICE`)
   - If no services appear, check that `apm_config.enabled: true` is set — navigate to **Infrastructure → Infrastructure List → your host** and confirm APM is listed under Apps
4. Use the **Environment** dropdown at the top of the Service Catalog to filter by your environment tag (e.g. `env:prod`)
5. Click your Nginx service name to open the **Service Overview Page**
6. On the Service Overview Page, identify and read each section:
   - **Requests graph** — total requests per second
   - **Errors graph** — error rate
   - **Latency graph** — p50, p75, p95, p99 percentiles
   - **Apdex score** (if enabled)
7. Change the time range (top right) to **Past 1 Hour**, then **Past 4 Hours**, and observe how graphs change
8. Click the **Resources** tab below the graphs
   - Resources are individual endpoints/routes (e.g. `GET /`, `POST /api/data`)
   - Note which resource has the highest request count

### ✅ Checkpoint

> 📝 Note the p95 latency for your top resource. You will set an alert on this in Lab 3.

---

## Part 3 — Exploring the Trace List (25 min)

**Navigation:** `APM → Traces`

### Steps

1. From the Service Overview Page, click **View Traces** (or go to **APM → Traces** in the sidebar)
2. On the Trace Explorer, set the search filter to your service:
   - In the search bar type: `service:nginx` (replace with your actual service name)
   - Press Enter
3. Observe the Trace List — each row is one request end-to-end
4. Click any trace row to open the **Trace Detail Panel**
5. In the Trace Detail Panel, explore:
   - **Flame Graph** — shows each span and its duration as a horizontal bar
   - **Span List** — text list of all spans in the trace
   - **Service Map** (if downstream services exist) — shows call chain
6. Click individual spans in the Flame Graph:
   - Note the **Resource Name** (e.g. `GET /`)
   - Note the **Duration**
   - Note **Metadata / Tags** on the right panel (`http.status_code`, `http.url`, etc.)
7. In the Trace Explorer search bar, add a filter for errors:
   - Type: `service:nginx status:error`
   - If no errors appear, remove the status filter and continue
8. Use the **Facets panel** on the left side to filter by:
   - **HTTP Status Code** — click `200` to see only successful requests
   - **Resource Name** — click your top endpoint to isolate it
9. Click **Save View** (top right of Trace Explorer) → name it `Nginx Overview` → Save

### ✅ Checkpoint

> 🔍 Open 3 different traces and compare their Flame Graphs. Identify which span takes the longest.

---

## Part 4 — Service Map Overview (25 min)

**Navigation:** `APM → Service Map`

### Steps

1. In the left sidebar, go to **APM → Service Map**
2. The Service Map renders all instrumented services as nodes connected by arrows
3. Locate your **Nginx node**
   - 🟢 **Green** = healthy
   - 🟡 **Yellow** = degraded
   - 🔴 **Red** = high error rate
4. Click the Nginx node:
   - A popup appears with Requests, Errors, and Latency mini-graphs
   - Click **View Service** to jump to the Service Overview Page
5. If your app has downstream dependencies (databases, APIs):
   - Follow the arrows from Nginx to those nodes
   - Click each downstream node and review its metrics
6. Use the **Environment** and **Time** filters at the top of the Service Map to change scope
7. Click **Inspect** on the Nginx node — this shows the inferred service dependencies Datadog detected automatically

### ✅ Checkpoint

> ✅ Lab 1 complete! You have verified agent health, explored the Service Catalog, Trace Explorer, and Service Map.

---

## Troubleshooting Tips

| Issue | What to check |
|---|---|
| Agent not showing green | Run `sudo datadog-agent status` — output shows exact failure reason |
| APM service not appearing | Verify `apm_config.enabled: true` in `/etc/datadog-agent/datadog.yaml` |
| Wrong service name in catalog | Check `DD_SERVICE` env var or `service:` tag in `datadog.yaml` |
| Flame graph looks flat | Normal for basic Nginx — deeper graphs appear with app-level tracing behind Nginx |
| No errors in trace filter | Run the 404 curl loop from Option C to generate error traces |
| No traces after 5+ minutes | Run `sudo datadog-agent status` — confirm Traces received > 0 |
| Nginx status 403 on `/nginx_status` | Confirm `allow 127.0.0.1` is in the location block and you're curling from localhost |
| `wrk` not found | Install with `sudo apt install wrk -y` or use `ab` instead |

---

## Quick Reference — Ubuntu Commands

```bash
# Nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl reload nginx            # apply config changes without downtime
sudo nginx -t                          # test config syntax before reloading
sudo tail -f /var/log/nginx/access.log

# Datadog Agent
sudo systemctl restart datadog-agent
sudo datadog-agent status              # full health check
sudo datadog-agent status | grep -A 15 "APM Agent"

# Load generation
ab -n 500 -c 10 http://localhost/      # quick burst
nohup ~/load_nginx.sh &                # continuous background load
pkill -f load_nginx.sh                 # stop background load
```

## Quick Reference — Datadog UI Navigation Paths

```
Agent Health         →  Infrastructure → Infrastructure List → [hostname]
Host Dashboard       →  Infrastructure → Infrastructure List → [hostname] → View Host Dashboard
Service Catalog      →  APM → Services
Service Overview     →  APM → Services → [service name]
Trace Explorer       →  APM → Traces
Service Map          →  APM → Service Map
API Keys             →  Organization Settings → API Keys
```

---

*Lab 1 of 5 · Datadog APM on Ubuntu + Nginx · All steps via Datadog UI*
