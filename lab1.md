# Datadog Hands-On Lab — Day 1
**Platform Deep Dive · Ubuntu 22.04 LTS**

> **Duration:** ~3 hours · Self-paced  
> **Environment:** Linux VM with sudo access

---

## Prerequisites

| Requirement | Detail |
|---|---|
| Linux VM | Ubuntu 22.04 / RHEL 8+ / Debian 12 |
| Internet Access | Outbound HTTPS to `datadoghq.com` |
| Datadog Account | Free trial at [app.datadoghq.com](https://app.datadoghq.com) |
| sudo Privileges | Root or sudoer on the VM |
| Python 3.8+ | For custom check exercises |

---

## Data Flow

```
Linux Host  →  DD Agent (v7)  →  HTTPS/443  →  Datadog Backend  →  Dashboards & Alerts
```

---

## Module 1 — Platform Overview
**⏱ 45 min · Browser**

### Key Concepts

| Term | What It Is |
|---|---|
| `DD_API_KEY` | 32-char key for sending data (metrics, logs, traces) |
| `DD_APP_KEY` | 40-char key for querying the management API |
| `DD_SITE` | Your account region endpoint (e.g. `datadoghq.com`) |
| Tags | `key:value` labels attached to every metric, log, and trace |
| Integration | Pre-built connector for a technology (nginx, postgres, etc.) |
| Custom Check | Python script the Agent runs to collect your own metrics |

### Account Setup

1. Sign up at [app.datadoghq.com](https://app.datadoghq.com) — select **US1** region
2. Go to **Organization Settings → API Keys → New Key**, name it `lab-workshop`
3. Go to **Organization Settings → Application Keys → New Key**, name it `lab-app-key`

**Test your API key from the VM:**
```bash
export DD_API_KEY=YOUR_API_KEY_HERE

curl -X POST "https://api.datadoghq.com/api/v1/validate" \
  -H "DD-API-KEY: ${DD_API_KEY}"
# Expected response: {"valid": true}
```

**Set environment variables for this session:**
```bash
export DD_API_KEY=<your-api-key>
export DD_APP_KEY=<your-app-key>
export DD_SITE=datadoghq.com
```

### UI Navigation Reference

> 💡 Use `Ctrl+K` / `⌘K` for global search anywhere in the UI.

| Section | Path | What to Explore |
|---|---|---|
| Host Map | Infrastructure → Host Map | All reporting hosts, color-coded by metric |
| Metrics Explorer | Metrics → Explorer | Query and graph any metric in real time |
| Dashboards | Dashboards → Dashboard List | OOTB and custom dashboards |
| Log Explorer | Logs → Search | Full-text search, facets, live tail |
| Monitors | Monitors → Manage | All alert rules grouped by status |
| Integrations | Integrations → Integrations | 850+ pre-built tiles |

---

## Module 2 — Unified Agent
**⏱ 60 min · Linux CLI**

> The **Unified Agent v7** is a single binary that collects metrics, logs, traces, processes, and security events.

### Step 1 — Install the Agent

**One-liner (recommended):**
```bash
DD_API_KEY=YOUR_API_KEY DD_SITE=datadoghq.com \
  bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script_agent7.sh)"
```

**Manual install (Debian/Ubuntu):**
```bash
sudo apt-get install -y apt-transport-https curl gnupg
sudo sh -c "echo 'deb [signed-by=/usr/share/keyrings/datadog-archive-keyring.gpg] \
  https://apt.datadoghq.com/ stable 7' > /etc/apt/sources.list.d/datadog.list"
sudo touch /usr/share/keyrings/datadog-archive-keyring.gpg
curl https://keys.datadoghq.com/DATADOG_APT_KEY_CURRENT.public \
  | sudo gpg --no-default-keyring \
  --keyring /usr/share/keyrings/datadog-archive-keyring.gpg --import
sudo apt-get update
sudo apt-get install -y datadog-agent
```

### Step 2 — Verify the Agent

```bash
sudo systemctl status datadog-agent
sudo datadog-agent status
sudo datadog-agent health
# Expected: Agent health: PASS
```

### Step 3 — Configure `/etc/datadog-agent/datadog.yaml`

```bash
# View active settings (non-comment lines)
sudo cat /etc/datadog-agent/datadog.yaml | grep -v "^#" | grep -v "^$"
```

Minimal config:
```yaml
api_key: YOUR_DD_API_KEY
site: datadoghq.com
hostname: my-linux-lab
tags:
  - env:lab
  - team:platform
  - workshop:datadog

logs_enabled: true

process_config:
  process_collection:
    enabled: true

apm_config:
  enabled: true
```

Restart after any config change:
```bash
sudo systemctl restart datadog-agent
```

### Step 4 — Explore Agent Directories

```bash
ls /etc/datadog-agent/        # datadog.yaml  conf.d/  checks.d/
ls /etc/datadog-agent/conf.d/ # per-integration config folders
ls /var/log/datadog/          # agent.log  trace-agent.log  process-agent.log
```

### Step 5 — Write a Custom Check

**`/etc/datadog-agent/checks.d/active_users.py`**
```python
from datadog_checks.base import AgentCheck
import random

class ActiveUsersCheck(AgentCheck):
    def check(self, instance):
        active_users = random.randint(100, 500)
        self.gauge(
            "app.active_users",
            active_users,
            tags=["env:lab", "service:myapp"]
        )
```

**`/etc/datadog-agent/conf.d/active_users.d/conf.yaml`**
```yaml
init_config:

instances:
  - {}
```

**Test the check:**
```bash
sudo -u dd-agent datadog-agent check active_users
sudo systemctl restart datadog-agent
```

### Module 2 Checklist

- [ ] Agent installed and running (`systemctl status datadog-agent`)
- [ ] `datadog-agent health` returns `PASS`
- [ ] Host visible in **Infrastructure → Host Map**
- [ ] Tags `env:lab` and `team:platform` visible in UI
- [ ] Custom check `app.active_users` reporting in Metrics Explorer

---

## Module 3 — Observability Hub
**⏱ 60 min · Linux CLI + Browser**

### Step 1 — Generate Infrastructure Load

```bash
sudo apt-get install -y stress-ng

# Stress CPU for 2 minutes
stress-ng --cpu 2 --timeout 120s &

# Stress memory
stress-ng --vm 1 --vm-bytes 512M --timeout 120s &

# Watch live metrics on the VM
watch -n2 "cat /proc/loadavg && free -h"
```

### Step 2 — Create a CPU Monitor

Go to **Monitors → New Monitor → Metric**:

| Setting | Value |
|---|---|
| Metric | `system.cpu.user` |
| From | `host:my-linux-lab` |
| Alert threshold | `> 70` |
| Warning threshold | `> 50` |
| Monitor name | `[Lab] High CPU on my-linux-lab` |
| Notification | Your email address |

### Step 3 — Instrument a Flask App with APM

**Install dependencies:**
```bash
pip3 install ddtrace flask
mkdir -p ~/lab-app && cd ~/lab-app
```

**`~/lab-app/app.py`**
```python
from flask import Flask, jsonify
import time, random

app = Flask(__name__)

@app.route("/api/users")
def get_users():
    time.sleep(random.uniform(0.01, 0.2))
    return jsonify({"users": 42, "status": "ok"})

@app.route("/api/orders")
def get_orders():
    time.sleep(random.uniform(0.05, 0.5))
    if random.random() < 0.1:
        raise Exception("Database timeout!")
    return jsonify({"orders": 128, "status": "ok"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**Run with APM and generate traffic:**
```bash
DD_SERVICE=lab-flask-app DD_ENV=lab DD_VERSION=1.0 \
  ddtrace-run python3 app.py &

for i in {1..100}; do
  curl -s http://localhost:5000/api/users > /dev/null
  curl -s http://localhost:5000/api/orders > /dev/null
  sleep 0.5
done
```

View traces at **APM → Services → lab-flask-app**.

### Step 4 — Build a Unified Dashboard

Go to **Dashboards → New Dashboard → Blank Dashboard** and add:

| Widget | Metric / Source | Title |
|---|---|---|
| Timeseries | `system.cpu.user{host:my-linux-lab}` | CPU Utilization % |
| Timeseries | `system.mem.used{host:my-linux-lab}` | Memory Used |
| Query Value | `avg:system.load.1{host:my-linux-lab}` | Load Average (1m) |
| APM Service Summary | `service:lab-flask-app env:lab` | Flask App P99 Latency |
| Monitor Summary | `env:lab` | Lab Monitor Status |

### Module 3 Checklist

- [ ] CPU/Memory stress test visible in **Metrics Explorer**
- [ ] CPU monitor created and showing OK/WARN state
- [ ] Flask app traces visible in **APM → Service Map**
- [ ] Unified dashboard created with all widgets

---

## Quick Reference — Agent Commands

```bash
sudo systemctl start datadog-agent      # Start
sudo systemctl stop datadog-agent       # Stop
sudo systemctl restart datadog-agent    # Restart
sudo datadog-agent status               # Full status report
sudo datadog-agent health               # Health check
sudo datadog-agent check <check_name>   # Test a specific check
sudo datadog-agent configcheck          # Validate config files
tail -f /var/log/datadog/agent.log      # Live agent logs
```

---

*Datadog Hands-On Lab Workshop v2.0 · Day 1*
