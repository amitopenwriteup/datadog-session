# Datadog Agent on EC2 — Full Lab Workshop

**Platform:** Amazon Linux 2 / Ubuntu  
**Duration:** ~90 minutes  
**Prerequisite:** Datadog Agent already installed

---

## Table of Contents

1. [Lab 1 — Verify the installation](#lab-1--verify-the-installation)
2. [Lab 2 — Configure datadog.yaml](#lab-2--configure-datadogyaml)
3. [Lab 3 — Set hostname & tags](#lab-3--set-hostname--tags)
4. [Lab 4 — Enable log collection](#lab-4--enable-log-collection)
5. [Lab 5 — Enable APM / tracing](#lab-5--enable-apm--tracing)
6. [Lab 6 — Enable process monitoring](#lab-6--enable-process-monitoring)
7. [Lab 7 — Write a custom check](#lab-7--write-a-custom-check)
8. [Lab 8 — Final validation](#lab-8--final-validation)

---

## Lab 1 — Verify the installation

**Goal:** Confirm the agent is running and locate all key config paths.

### Step 1 — Check service status and version

```bash
# Service status
sudo systemctl status datadog-agent

# Agent version
sudo datadog-agent version
```

### Step 2 — Locate config paths

```bash
ls /etc/datadog-agent/
ls /etc/datadog-agent/conf.d/
```

### Step 3 — Run full agent status

```bash
sudo datadog-agent status
```

> **Expected:** "Running" under Agent Status, zero errors in the Collector section.

---

## Lab 2 — Configure datadog.yaml

**Goal:** Set the API key, site, and core agent settings.

### Step 1 — Open the main config file

```bash
sudo vi /etc/datadog-agent/datadog.yaml
```

### Step 2 — Set API key and site

Find and uncomment these lines:

```yaml
api_key: "<YOUR_DD_API_KEY>"
site: "datadoghq.com"   # EU users: datadoghq.eu
```

> Get your API key from **Datadog → Organization Settings → API Keys**.

### Step 3 — Validate config syntax

Always run this before restarting:

```bash
sudo datadog-agent configcheck
```

> **Warning:** A bad YAML config silently disables the agent. Fix all errors before proceeding.

### Step 4 — Restart the agent

```bash
sudo systemctl restart datadog-agent
sudo systemctl status datadog-agent
```

---

## Lab 3 — Set hostname & tags

**Goal:** Tag your host for filtering and scoping in Datadog dashboards and monitors.

### Step 1 — Add hostname and tags in datadog.yaml

```yaml
hostname: "ec2-lab-host"
env: "lab"

tags:
  - env:lab
  - team:platform
  - region:us-east-1
  - cloud:aws
```

### Step 2 — Enable automatic EC2 tags from instance metadata

```yaml
collect_ec2_tags: true
collect_ec2_tags_use_imds: true
```

> This pulls EC2 tags (Name, Environment, etc.) automatically via the Instance Metadata Service. No extra IAM credentials needed if the instance role has `ec2:DescribeTags`.

### Step 3 — Restart and verify

```bash
sudo systemctl restart datadog-agent
sudo datadog-agent status | grep -A 10 "Host Tags"
```

---

## Lab 4 — Enable log collection

**Goal:** Stream syslog and auth.log to Datadog Log Management.

### Step 1 — Enable logs globally in datadog.yaml

```yaml
logs_enabled: true

logs_config:
  container_collect_all: false
  use_compression: true
  compression_level: 6
```

### Step 2 — Create a log collection config

```bash
sudo mkdir -p /etc/datadog-agent/conf.d/linux_log.d

sudo tee /etc/datadog-agent/conf.d/linux_log.d/conf.yaml > /dev/null <<EOF
logs:
  - type: file
    path: /var/log/syslog
    service: syslog
    source: linux
    tags:
      - env:lab

  - type: file
    path: /var/log/auth.log
    service: auth
    source: linux
    tags:
      - env:lab
EOF
```

> **Amazon Linux 2 note:** Replace `/var/log/syslog` with `/var/log/messages`.

### Step 3 — Grant dd-agent user read access

```bash
sudo usermod -a -G adm dd-agent
sudo chmod g+r /var/log/syslog /var/log/auth.log
```

### Step 4 — Restart and confirm logs pipeline

```bash
sudo systemctl restart datadog-agent
sudo datadog-agent status | grep -A 5 "Logs Agent"
```

> **Expected:** "Logs Agent running" with `bytes_sent` incrementing.

---

## Lab 5 — Enable APM / tracing

**Goal:** Start the trace receiver so instrumented apps can send spans to Datadog APM.

### Step 1 — Enable APM in datadog.yaml

```yaml
apm_config:
  enabled: true
  env: "lab"
  receiver_port: 8126
  apm_non_local_traffic: false
```

### Step 2 — Restart and verify the trace receiver is listening

```bash
sudo systemctl restart datadog-agent
sudo ss -tlnp | grep 8126
```

> **Expected:** `LISTEN 0 128 127.0.0.1:8126`

### Step 3 — Send a test trace to confirm end-to-end

```bash
curl -X POST http://localhost:8126/v0.4/traces \
  -H 'Content-Type: application/json' \
  -d '[[{
    "name": "test.span",
    "service": "lab-test",
    "resource": "/test",
    "trace_id": 1,
    "span_id": 1,
    "parent_id": 0,
    "start": 1000000,
    "duration": 500000,
    "error": 0,
    "meta": {},
    "metrics": {},
    "type": "web"
  }]]'
```

> Check Datadog APM → Traces for `service:lab-test` within a few seconds.

---

## Lab 6 — Enable process monitoring

**Goal:** Collect live process data (CPU, memory, command line) from the EC2 instance.

### Step 1 — Add process collection in datadog.yaml

```yaml
process_config:
  process_collection:
    enabled: true
  container_collection:
    enabled: false
  scrub_args: true
  custom_sensitive_words:
    - password
    - token
    - secret
```

> `scrub_args: true` masks sensitive CLI arguments (passwords, tokens) before sending to Datadog.

### Step 2 — Restart and verify process agent

```bash
sudo systemctl restart datadog-agent
sudo datadog-agent status | grep -A 5 "Process Agent"
```

> **Expected:** "Process Agent running" with process count reported.

---

## Lab 7 — Write a custom check

**Goal:** Emit a custom metric and service check from a Python check script.

### Step 1 — Create the check Python file

```bash
sudo tee /etc/datadog-agent/checks.d/lab_check.py > /dev/null <<'EOF'
from datadog_checks.base import AgentCheck
import random

class LabCheck(AgentCheck):
    def check(self, instance):
        # Emit a gauge metric with a random value
        self.gauge(
            'lab.custom.metric',
            random.uniform(0, 100),
            tags=['env:lab', 'check:lab_check']
        )
        # Emit a service check
        self.service_check(
            'lab.custom.ok',
            AgentCheck.OK,
            tags=['env:lab']
        )
EOF
```

### Step 2 — Create the check config file

```bash
sudo mkdir -p /etc/datadog-agent/conf.d/lab_check.d

sudo tee /etc/datadog-agent/conf.d/lab_check.d/conf.yaml > /dev/null <<EOF
init_config:

instances:
  - min_collection_interval: 30
EOF
```

### Step 3 — Test the check in isolation

Run the check without restarting the agent:

```bash
sudo -u dd-agent datadog-agent check lab_check
```

> **Expected output:** `lab.custom.metric` with a random float value, and service check status `OK`.

### Step 4 — Load into the running agent

```bash
sudo systemctl restart datadog-agent
```

> Verify in Datadog: **Metrics Explorer → search `lab.custom.metric`**

---

## Lab 8 — Final validation

**Goal:** Confirm all labs are working, connectivity is healthy, and the host is visible in Datadog.

### Step 1 — Full status check with error filter

```bash
sudo datadog-agent status 2>&1 | tee /tmp/dd-status.txt
grep -iE "error|failed|warning" /tmp/dd-status.txt
```

> Zero critical errors expected. Warnings about optional features are acceptable.

### Step 2 — Connectivity diagnostics

```bash
sudo datadog-agent diagnose \
  --include connectivity-datadog-core-endpoints
```

### Step 3 — Generate a flare bundle

If you need support escalation, generate a flare — it bundles all logs, config (with secrets redacted), and status output:

```bash
sudo datadog-agent flare
```

### Step 4 — Verify in the Datadog UI

| What to check | Where to look |
|---|---|
| Host visible | Infrastructure → Host Map → search hostname |
| Tags applied | Infrastructure → Host Map → host detail panel |
| Custom metric | Metrics Explorer → `lab.custom.metric` |
| Syslog ingested | Logs → `service:syslog env:lab` |
| APM trace | APM → Traces → `service:lab-test` |
| Processes | Infrastructure → Processes |

---

## Quick reference

### Key file paths

| File | Path |
|---|---|
| Main config | `/etc/datadog-agent/datadog.yaml` |
| Integration checks | `/etc/datadog-agent/conf.d/<check>.d/conf.yaml` |
| Custom check scripts | `/etc/datadog-agent/checks.d/<check>.py` |
| Agent logs | `/var/log/datadog/agent.log` |

### Essential CLI commands

| Command | Purpose |
|---|---|
| `sudo datadog-agent status` | Full runtime status |
| `sudo datadog-agent configcheck` | Validate YAML before restart |
| `sudo datadog-agent check <name>` | Run a single check in isolation |
| `sudo datadog-agent diagnose` | Connectivity and config diagnostics |
| `sudo datadog-agent flare` | Bundle logs + config for support |
| `sudo systemctl restart datadog-agent` | Restart the agent service |

### Environment variable overrides

Any `datadog.yaml` key can be overridden with `DD_<KEY>`:

```bash
export DD_API_KEY="your-key"
export DD_SITE="datadoghq.com"
export DD_LOGS_ENABLED="true"
export DD_APM_ENABLED="true"
export DD_ENV="lab"
export DD_TAGS="team:platform,region:us-east-1"
```

---

*Datadog Agent EC2 Lab Workshop — generated for Amazon Linux 2 / Ubuntu with Agent v7+*
