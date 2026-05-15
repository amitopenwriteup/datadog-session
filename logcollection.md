## Step 2 — Create log collection config

Create the directory:

```bash id="5y3xkq"
sudo mkdir -p /etc/datadog-agent/conf.d/linux_log.d
```

Create the Datadog log config file:

```bash id="g4n1dw"
sudo tee /etc/datadog-agent/conf.d/linux_log.d/conf.yaml > /dev/null <<'EOF'
logs:
  - type: file
    path: /var/log/syslog
    service: linux-system
    source: syslog

  - type: file
    path: /var/log/auth.log
    service: linux-auth
    source: linux
EOF
```

---

## Step 3 — Grant `dd-agent` read access

Add the Datadog agent user to the `adm` group:

```bash id="7m2sra"
sudo usermod -aG adm dd-agent
```

Grant group read permissions to the log files:

```bash id="k8w4zu"
sudo chmod g+r /var/log/syslog /var/log/auth.log
```

---

## Step 4 — Restart and verify Logs Agent

Restart the Datadog agent:

```bash id="a1q9pl"
sudo systemctl restart datadog-agent
```

Verify the Logs Agent status:

```bash id="r6v3tn"
sudo datadog-agent status | grep -A10 "Logs Agent"
```

You should see something similar to:

```text id="m4d7yb"
Logs Agent
==========
  Running: true
  Inputs:
    /var/log/syslog
    /var/log/auth.log
```

To confirm logs in UI:

1. Open [Datadog Logs Explorer](https://app.datadoghq.com/logs?utm_source=chatgpt.com)
2. Search:

```text id="0e9xsz"
service:linux-system
```

or

```text id="q2b8nv"
source:syslog
```
