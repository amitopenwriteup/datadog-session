# AWS × Datadog Integration Workshop
### UI-Based Hands-On Lab Guide

---

> **Lab Type:** Console/UI Driven — No CLI, No Automation  
> **Difficulty:** Intermediate  
> **Estimated Duration:** 3–4 Hours  
> **Prerequisites:** Active AWS Account · Active Datadog Trial/Account · Admin Access to Both

---

## Table of Contents

1. [Lab Overview](#1-lab-overview)
2. [Architecture Diagram](#2-architecture-diagram)
3. [Lab Environment Setup](#3-lab-environment-setup)
4. [Module 1 — Connect AWS to Datadog via IAM Role](#module-1--connect-aws-to-datadog-via-iam-role)
5. [Module 2 — Enable AWS Services in Datadog](#module-2--enable-aws-services-in-datadog)
6. [Module 3 — Set Up the Datadog Agent on EC2](#module-3--set-up-the-datadog-agent-on-ec2)
7. [Module 4 — CloudWatch Logs Forwarding to Datadog](#module-4--cloudwatch-logs-forwarding-to-datadog)
8. [Module 5 — Create Monitors and Alerts](#module-5--create-monitors-and-alerts)
9. [Module 6 — Build a Dashboard](#module-6--build-a-dashboard)
10. [Module 7 — RDS Integration](#module-7--rds-integration)
11. [Module 8 — S3 and Lambda Visibility](#module-8--s3-and-lambda-visibility)
12. [Validation Checklist](#validation-checklist)
13. [Troubleshooting Guide](#troubleshooting-guide)
14. [Lab Teardown](#lab-teardown)

---

## 1. Lab Overview

This workshop walks you through integrating **Amazon Web Services (AWS)** with **Datadog** using only web-based consoles — no terminal, no scripts, no IaC. Every step is performed through:

- **AWS Management Console** (console.aws.amazon.com)
- **Datadog Web UI** (app.datadoghq.com)

### What You Will Accomplish

| # | Objective |
|---|-----------|
| 1 | Establish a secure AWS-Datadog connection using an IAM Role |
| 2 | Pull EC2, RDS, S3, Lambda, and ELB metrics into Datadog |
| 3 | Forward CloudWatch logs to Datadog |
| 4 | Install and configure the Datadog Agent on an EC2 instance |
| 5 | Create Monitors with threshold-based alerts |
| 6 | Build an operational dashboard for your AWS environment |

### Key Concepts

- **IAM Role (Cross-Account):** Datadog uses a cross-account IAM role with a specific External ID — this is more secure than access keys.
- **AWS Integration Tile:** Datadog's UI for selecting which AWS services and regions to monitor.
- **Datadog Agent:** A lightweight process installed on EC2 to collect system-level and application metrics.
- **Log Forwarder Lambda:** A Datadog-managed Lambda function that ships CloudWatch logs to Datadog.

---

## 2. Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                      AWS Account                        │
│                                                         │
│  ┌─────────┐   ┌──────────┐   ┌───────┐   ┌────────┐  │
│  │   EC2   │   │   RDS    │   │  S3   │   │Lambda  │  │
│  │ Instance│   │ Database │   │Bucket │   │Function│  │
│  └────┬────┘   └────┬─────┘   └───┬───┘   └───┬────┘  │
│       │             │             │            │        │
│  ┌────▼─────────────▼─────────────▼────────────▼─────┐ │
│  │               CloudWatch                          │ │
│  │         (Metrics + Logs + Events)                 │ │
│  └───────────────────────┬────────────────────────────┘ │
│                          │                              │
│  ┌───────────────────────▼───────────────────────────┐  │
│  │        IAM Role: DatadogIntegrationRole           │  │
│  │        (ReadOnly + SecurityAudit Policies)        │  │
│  └───────────────────────┬───────────────────────────┘  │
└──────────────────────────┼──────────────────────────────┘
                           │  Secure Cross-Account Access
                           ▼
              ┌────────────────────────┐
              │      Datadog           │
              │  ┌──────────────────┐  │
              │  │ Metrics Explorer │  │
              │  │ Log Management   │  │
              │  │ Monitors/Alerts  │  │
              │  │ Dashboards       │  │
              │  └──────────────────┘  │
              └────────────────────────┘
```

---

## 3. Lab Environment Setup

### 3.1 AWS Prerequisites

1. Log in to **AWS Management Console**: https://console.aws.amazon.com
2. Ensure your user has the following permissions:
   - `IAMFullAccess` (to create roles)
   - `EC2FullAccess`
   - `RDSFullAccess`
   - `CloudWatchFullAccess`
   - `LambdaFullAccess`
3. Set your region to **us-east-1** (or your preferred region — keep it consistent).

### 3.2 Datadog Prerequisites

1. Log in to **Datadog**: https://app.datadoghq.com
2. Navigate to **Organization Settings → API Keys**.
3. Click **+ New Key** → Name it `workshop-key` → **Create Key**.
4. **Copy and save the API key** — you will need it multiple times.

> ⚠️ **Important:** Note your Datadog site (e.g., `datadoghq.com` or `datadoghq.eu`). All URLs in this lab use `datadoghq.com`. Adjust if your account is on a different site.

---

## Module 1 — Connect AWS to Datadog via IAM Role

**Goal:** Create a cross-account IAM Role that allows Datadog to read AWS metadata and metrics.

**Estimated time:** 20 minutes

---

### Step 1.1 — Get the Datadog AWS Account ID and External ID

1. In Datadog, go to **Integrations → Amazon Web Services**.
2. Click **Add AWS Account**.
3. Choose **Automatically using CloudFormation** — then **switch to Manual**.
4. On the Manual configuration screen, note:
   - **Datadog AWS Account ID** (e.g., `464622532012`)
   - **External ID** — Copy this value. It is unique to your Datadog org.

> Keep this tab open — you will return to it.

---

### Step 1.2 — Create the IAM Role in AWS

1. Open a new tab: **AWS Console → IAM → Roles**.
2. Click **Create role**.
3. Under **Trusted entity type**, select **AWS account**.
4. Select **Another AWS account**.
5. In the **Account ID** field, enter Datadog's AWS Account ID from Step 1.1.
6. Check **Require external ID**.
7. In the **External ID** field, paste the External ID from Step 1.1.
8. Click **Next**.

---

### Step 1.3 — Attach Permissions to the Role

On the **Add permissions** screen, search for and attach these policies (check each one):

| Policy Name | Purpose |
|-------------|---------|
| `SecurityAudit` | Read security and compliance info |
| `AmazonEC2ReadOnlyAccess` | Read EC2 instance metadata |
| `CloudWatchReadOnlyAccess` | Read CloudWatch metrics |
| `AmazonRDSReadOnlyAccess` | Read RDS metadata |
| `AmazonS3ReadOnlyAccess` | Read S3 bucket metadata |
| `AWSLambdaReadOnlyAccess` | Read Lambda metadata |
| `ElasticLoadBalancingReadOnly` | Read ELB metadata |

After selecting all seven, click **Next**.

---

### Step 1.4 — Name and Create the Role

1. **Role name:** `DatadogIntegrationRole`
2. **Description:** `Allows Datadog to read AWS metrics and metadata`
3. Add a tag:
   - **Key:** `CreatedBy`
   - **Value:** `DatadogWorkshop`
4. Click **Create role**.
5. Open the newly created role and copy the **ARN** (e.g., `arn:aws:iam::123456789012:role/DatadogIntegrationRole`).

---

### Step 1.5 — Complete the Integration in Datadog

1. Return to the Datadog **Add AWS Account** tab.
2. In the **AWS Role ARN** field, paste the ARN you copied.
3. Leave **Enable Resource Collection** checked.
4. Click **Save**.

> ✅ **Expected Result:** After 2–5 minutes, you should see a green checkmark next to your AWS account in the Datadog AWS Integration page.

---

## Module 2 — Enable AWS Services in Datadog

**Goal:** Choose which AWS services and regions Datadog should monitor.

**Estimated time:** 10 minutes

---

### Step 2.1 — Configure the Integration Settings

1. In Datadog, go to **Integrations → Amazon Web Services**.
2. Click on your newly added AWS account.
3. Go to the **Metric Collection** tab.

---

### Step 2.2 — Select Services to Monitor

Enable the following services by toggling them ON:

- ✅ EC2
- ✅ RDS
- ✅ S3
- ✅ Lambda
- ✅ ELB (Elastic Load Balancing)
- ✅ CloudFront
- ✅ SQS
- ✅ CloudTrail

> **Tip:** You can use the search box to find services quickly.

---

### Step 2.3 — Limit to Specific Regions

1. Under **Limit metric collection to specific resources**, select your region (e.g., `us-east-1`).
2. This prevents Datadog from pulling metrics from regions you don't use, reducing cost.

---

### Step 2.4 — Configure Tags (Optional but Recommended)

1. Go to the **Tags** tab within the AWS integration settings.
2. Add a global tag to all metrics from this account:
   - `env:workshop`
   - `team:platform`
3. Click **Save**.

> ✅ **Validation:** Navigate to **Metrics → Explorer** in Datadog. In the metric search box, type `aws.ec2` — you should start seeing metric names populate within 10 minutes.

---

## Module 3 — Set Up the Datadog Agent on EC2

**Goal:** Install the Datadog Agent on an EC2 instance to collect system-level metrics (CPU, memory, disk, network) and enable APM.

**Estimated time:** 30 minutes

---

### Step 3.1 — Launch an EC2 Instance

1. Go to **AWS Console → EC2 → Instances → Launch Instances**.
2. Configure:
   - **Name:** `datadog-workshop-server`
   - **AMI:** Amazon Linux 2023 (64-bit)
   - **Instance Type:** `t3.micro`
   - **Key Pair:** Create new → Name `workshop-keypair` → Download `.pem`
   - **Security Group:** Create new → Name `datadog-workshop-sg`
     - Allow **SSH (port 22)** from your IP
     - Allow **HTTP (port 80)** from anywhere
3. Under **Advanced Details → User Data**, paste the following:

```
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Datadog Workshop Server</h1>" > /var/www/html/index.html
```

4. Click **Launch Instance**.
5. Wait for the instance **State** to show `Running` and **Status Checks** to show `2/2 checks passed`.

---

### Step 3.2 — Connect to the Instance via EC2 Instance Connect

1. Select your instance in the EC2 console.
2. Click **Connect** (top right).
3. Choose **EC2 Instance Connect** tab.
4. Click **Connect** — a browser-based terminal opens.

> This is a UI-based terminal — no local SSH client required.

---

### Step 3.3 — Install the Datadog Agent via the UI Terminal

In the EC2 Instance Connect terminal, run the one-line install command. To get this command:

1. In Datadog, go to **Integrations → Agent → Amazon Linux**.
2. Copy the install command shown — it looks like:

```
DD_API_KEY=<your-api-key> DD_SITE="datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

3. Replace `<your-api-key>` with your API key from Section 3.2.
4. Paste and run in the Instance Connect terminal.

> The install takes about 60–90 seconds.

---

### Step 3.4 — Verify Agent Is Running

In the Instance Connect terminal, run:

```
sudo systemctl status datadog-agent
```

You should see `active (running)` in green.

---

### Step 3.5 — Add Tags to the Agent

1. In the Instance Connect terminal, run:

```
sudo nano /etc/datadog-agent/datadog.yaml
```

2. Find the `tags:` section (scroll down or use `Ctrl+W` to search).
3. Add:

```yaml
tags:
  - env:workshop
  - role:webserver
  - team:platform
```

4. Press `Ctrl+X`, then `Y`, then `Enter` to save.
5. Restart the agent:

```
sudo systemctl restart datadog-agent
```

---

### Step 3.6 — Verify in Datadog

1. In Datadog, go to **Infrastructure → Hosts**.
2. Your EC2 instance should appear within 2–3 minutes.
3. Click on the host to see CPU, memory, disk, and network metrics.

> ✅ **Expected Result:** Host visible in Infrastructure with system metrics populating.

---

## Module 4 — CloudWatch Logs Forwarding to Datadog

**Goal:** Forward CloudWatch logs to Datadog using the Datadog Forwarder Lambda function.

**Estimated time:** 30 minutes

---

### Step 4.1 — Deploy the Datadog Forwarder via CloudFormation

1. Go to **AWS Console → CloudFormation → Create Stack → With new resources**.
2. Select **Template is ready → Amazon S3 URL**.
3. Paste this URL:

```
https://datadog-cloudformation-template.s3.amazonaws.com/aws/forwarder/latest.yaml
```

4. Click **Next**.

---

### Step 4.2 — Configure Stack Parameters

| Parameter | Value |
|-----------|-------|
| Stack Name | `datadog-forwarder` |
| DdApiKey | Your Datadog API key |
| DdSite | `datadoghq.com` |
| FunctionName | `datadog-forwarder` |

Leave all other parameters at defaults.

5. Click **Next → Next → Check the IAM acknowledgement box → Submit**.
6. Wait for stack status to become `CREATE_COMPLETE` (about 3–5 minutes).

---

### Step 4.3 — Subscribe CloudWatch Log Group to the Forwarder

1. Go to **AWS Console → CloudWatch → Log Groups**.
2. Find the log group `/var/log/httpd/access_log` — if it doesn't exist yet, select `/aws/lambda/` or any existing log group.
3. Click on the log group name.
4. Go to **Subscription Filters → Create Lambda subscription filter**.
5. Configure:
   - **Lambda Function:** Select `datadog-forwarder`
   - **Log format:** Other
   - **Subscription filter name:** `datadog-forward`
6. Click **Start streaming**.

---

### Step 4.4 — Generate Some Logs

1. Go back to EC2, find your instance's **Public IPv4 DNS**.
2. Open it in a browser — you should see the Apache welcome page.
3. Refresh the page 10–15 times to generate access logs.

---

### Step 4.5 — Verify Logs in Datadog

1. In Datadog, go to **Logs → Search**.
2. In the search bar, enter `source:apache` or `host:<your-ec2-hostname>`.
3. You should see log entries appearing within 1–2 minutes.

> ✅ **Expected Result:** Apache access logs visible in Datadog Log Management.

---

## Module 5 — Create Monitors and Alerts

**Goal:** Set up threshold-based monitors to alert when AWS resources are under stress.

**Estimated time:** 25 minutes

---

### Step 5.1 — Create a CPU Monitor for EC2

1. In Datadog, go to **Monitors → New Monitor**.
2. Select **Metric**.
3. Configure the monitor:

**Step 1 — Define the metric:**
- Metric: `system.cpu.user`
- From: `host:<your-ec2-instance-id>`
- Aggregation: `avg by host`

**Step 2 — Set alert conditions:**
- Alert threshold: `> 80`
- Warning threshold: `> 60`
- Evaluation window: `Last 5 minutes`

**Step 3 — Notify your team:**
- Monitor name: `[EC2] High CPU Usage on Workshop Server`
- Message body:
```
{{#is_alert}}
⚠️ HIGH CPU ALERT
Host: {{host.name}}
Current CPU: {{value}}%
Threshold: 80%

Check the EC2 console immediately.
{{/is_alert}}

{{#is_recovery}}
✅ CPU has recovered below threshold.
{{/is_recovery}}
```
- Tags: `env:workshop`, `service:ec2`

4. Click **Create Monitor**.

---

### Step 5.2 — Create an RDS Connection Count Monitor

1. Go to **Monitors → New Monitor → Metric**.
2. Configure:
   - Metric: `aws.rds.database_connections`
   - Alert threshold: `> 100`
   - Warning threshold: `> 80`
   - Name: `[RDS] High Connection Count`
3. Click **Create Monitor**.

---

### Step 5.3 — Create a Log-Based Monitor for HTTP Errors

1. Go to **Monitors → New Monitor → Logs**.
2. Configure:
   - Search query: `source:apache status:error`
   - Evaluation window: `5 minutes`
   - Alert threshold: `> 10` errors in 5 minutes
   - Name: `[Apache] Elevated HTTP Error Rate`
3. Click **Create Monitor**.

---

### Step 5.4 — Create a Monitor Group (Multi Alert)

1. Go to **Monitors → New Monitor → Metric**.
2. Configure:
   - Metric: `aws.ec2.network_in`
   - Group by: `host`
   - This creates one alert per EC2 host automatically.
   - Name: `[EC2] Network In — by Host`
3. Click **Create Monitor**.

> ✅ **Validation:** Go to **Monitors → Manage Monitors** — you should see all four monitors listed with status `OK` or `No Data`.

---

## Module 6 — Build a Dashboard

**Goal:** Create a comprehensive AWS Operations Dashboard in Datadog.

**Estimated time:** 30 minutes

---

### Step 6.1 — Create a New Dashboard

1. Go to **Dashboards → New Dashboard**.
2. Name it: `AWS Workshop — Operations Overview`.
3. Select **New Dashboard** (free layout).
4. Click **Create Dashboard**.

---

### Step 6.2 — Add a Summary Header Row

**Widget 1 — Query Value: EC2 Instance Count**
1. Click **Add Widget → Query Value**.
2. Metric: `aws.ec2.host_ok`
3. Title: `EC2 Instances`
4. Click **Save**.

**Widget 2 — Query Value: Active RDS Connections**
1. Add another **Query Value**.
2. Metric: `aws.rds.database_connections`
3. Aggregation: `avg`
4. Title: `RDS Connections`

**Widget 3 — Monitor Summary**
1. Add **Monitor Summary** widget.
2. Filter: `env:workshop`
3. Title: `Monitor Status`

---

### Step 6.3 — Add a Time-Series Row

**Widget 4 — EC2 CPU Over Time**
1. Add **Timeseries** widget.
2. Metric: `system.cpu.user`
3. Group by: `host`
4. Display as: `Lines`
5. Title: `EC2 CPU Usage (%)`

**Widget 5 — EC2 Memory Over Time**
1. Add **Timeseries** widget.
2. Metric: `system.mem.used`
3. Group by: `host`
4. Title: `EC2 Memory Used`

**Widget 6 — Network I/O**
1. Add **Timeseries** widget.
2. Add two metrics:
   - `aws.ec2.network_in` — Label: `Network In`
   - `aws.ec2.network_out` — Label: `Network Out`
3. Title: `EC2 Network I/O`

---

### Step 6.4 — Add a Logs Widget

1. Add **Log Stream** widget.
2. Query: `host:<your-ec2-hostname>`
3. Columns: `host`, `service`, `message`
4. Title: `Live Log Stream`
5. Click **Save**.

---

### Step 6.5 — Add a Top List Widget

1. Add **Top List** widget.
2. Metric: `system.disk.used_pct`
3. Group by: `host`
4. Title: `Disk Usage by Host`
5. Click **Save**.

---

### Step 6.6 — Arrange and Polish

1. Drag widgets to arrange them in rows:
   - **Row 1:** Summary metrics (3 query value widgets)
   - **Row 2:** CPU, Memory, Network timeseries
   - **Row 3:** Logs stream + Top list
2. Resize widgets by dragging their edges.
3. Click **Save Changes** (top right).

> ✅ **Expected Result:** A multi-panel dashboard showing live AWS metrics, host data, and logs.

---

## Module 7 — RDS Integration

**Goal:** Spin up an RDS instance and monitor it in Datadog.

**Estimated time:** 25 minutes

---

### Step 7.1 — Launch an RDS Instance

1. Go to **AWS Console → RDS → Create Database**.
2. Configure:
   - **Creation method:** Standard Create
   - **Engine:** MySQL
   - **Version:** MySQL 8.0.x
   - **Template:** Free Tier
   - **DB Instance Identifier:** `workshop-db`
   - **Master username:** `admin`
   - **Master password:** `Workshop123!`
   - **Instance class:** `db.t3.micro`
   - **Storage:** 20 GiB GP2
   - **VPC:** Default VPC
   - **Public access:** Yes (for lab only)
   - **Security Group:** Create new → `rds-workshop-sg`
     - Inbound: MySQL/Aurora (3306) from your EC2 security group
3. Under **Additional configuration → Initial database name:** `workshopdb`
4. Uncheck **Enable automated backups** (saves time in lab).
5. Click **Create Database**.

> This takes 5–10 minutes to become available.

---

### Step 7.2 — Enable Enhanced Monitoring

1. Once the RDS instance is `Available`, click on it.
2. Click **Modify**.
3. Under **Monitoring**, enable **Enhanced Monitoring**.
4. Set granularity to **60 seconds**.
5. Click **Continue → Apply immediately → Modify DB Instance**.

---

### Step 7.3 — View RDS Metrics in Datadog

1. In Datadog, go to **Metrics → Explorer**.
2. Search for `aws.rds`.
3. You should see metrics including:
   - `aws.rds.cpuutilization`
   - `aws.rds.database_connections`
   - `aws.rds.freeable_memory`
   - `aws.rds.read_iops`
   - `aws.rds.write_iops`

4. Add `aws.rds.cpuutilization` to your dashboard.

---

### Step 7.4 — Create RDS-Specific Dashboard Widget

1. Open your **AWS Workshop — Operations Overview** dashboard.
2. Add a new **Timeseries** widget.
3. Add these metrics:
   - `aws.rds.cpuutilization`
   - `aws.rds.freeable_memory`
4. Title: `RDS Performance`
5. Save.

---

## Module 8 — S3 and Lambda Visibility

**Goal:** Monitor S3 bucket size and Lambda function errors in Datadog.

**Estimated time:** 20 minutes

---

### Step 8.1 — Create an S3 Bucket with Request Metrics Enabled

1. Go to **AWS Console → S3 → Create Bucket**.
2. Configure:
   - **Bucket name:** `datadog-workshop-<your-initials>-<random>`
   - **Region:** us-east-1
   - Uncheck **Block all public access** (for lab visibility)
   - Acknowledge warning
3. Click **Create Bucket**.
4. Open the bucket → **Properties** tab.
5. Scroll to **Amazon S3 request metrics** → Click **Create filter**.
   - Filter name: `AllObjects`
   - Leave prefix empty (monitors all objects)
6. Click **Save**.

---

### Step 8.2 — Create a Lambda Function

1. Go to **AWS Console → Lambda → Create Function**.
2. Configure:
   - **Name:** `workshop-demo-function`
   - **Runtime:** Python 3.12
   - **Architecture:** x86_64
   - **Execution role:** Create a new role with basic Lambda permissions
3. Click **Create Function**.
4. In the **Code** tab, replace the default code with:

```python
import json
import random

def lambda_handler(event, context):
    # Simulate occasional errors for demo
    if random.random() < 0.2:
        raise Exception("Simulated error for Datadog demo")
    
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Hello from Workshop Lambda!'})
    }
```

5. Click **Deploy**.

---

### Step 8.3 — Create a Test Event and Invoke Several Times

1. Click **Test → Create new test event**.
2. Name: `workshop-test`
3. Keep the default JSON body.
4. Click **Save**.
5. Click **Test** 15–20 times (some will succeed, some will error, due to the random logic).

---

### Step 8.4 — View Lambda Metrics in Datadog

1. In Datadog → **Metrics Explorer**.
2. Search `aws.lambda`:
   - `aws.lambda.invocations`
   - `aws.lambda.errors`
   - `aws.lambda.duration`
3. Add a **Timeseries** widget to your dashboard:
   - Title: `Lambda Invocations and Errors`
   - Metrics: `aws.lambda.invocations` and `aws.lambda.errors`
4. Save dashboard.

---

### Step 8.5 — Create a Lambda Error Monitor

1. Go to **Monitors → New Monitor → Metric**.
2. Configure:
   - Metric: `aws.lambda.errors`
   - From: `functionname:workshop-demo-function`
   - Alert threshold: `> 3` in 5 minutes
   - Name: `[Lambda] Error Rate — workshop-demo-function`
3. Click **Create Monitor**.

---

## Validation Checklist

Use this checklist to confirm your lab is complete:

### Integration

- [ ] AWS account visible in **Integrations → Amazon Web Services** with ✅
- [ ] IAM Role `DatadogIntegrationRole` exists in AWS
- [ ] `aws.ec2.*` metrics appearing in **Metrics Explorer**

### Agent

- [ ] EC2 host visible in **Infrastructure → Hosts**
- [ ] System metrics (CPU, memory, disk) appearing for the host
- [ ] Host tagged with `env:workshop`

### Logs

- [ ] Datadog Forwarder Lambda deployed in CloudFormation
- [ ] CloudWatch subscription filter created
- [ ] Logs visible in **Logs → Search**

### Monitors

- [ ] `[EC2] High CPU Usage` monitor created
- [ ] `[RDS] High Connection Count` monitor created
- [ ] `[Apache] Elevated HTTP Error Rate` monitor created
- [ ] `[Lambda] Error Rate` monitor created

### Dashboard

- [ ] `AWS Workshop — Operations Overview` dashboard created
- [ ] EC2 CPU, Memory, Network widgets added
- [ ] Log stream widget added
- [ ] RDS Performance widget added
- [ ] Lambda widget added

### Services

- [ ] RDS instance `workshop-db` visible in Datadog
- [ ] Lambda function `workshop-demo-function` metrics visible
- [ ] S3 request metrics appearing

---

## Troubleshooting Guide

### Issue: AWS Integration shows red ✗ or "No Data"

**Possible Causes & Fixes:**

1. **Role ARN mismatch** — Go to IAM → Roles → `DatadogIntegrationRole`. Verify the ARN matches exactly what was entered in Datadog.
2. **External ID mismatch** — In IAM, click the role → Trust relationships → Edit trust policy. Verify `sts:ExternalId` matches Datadog's External ID.
3. **Missing permissions** — Check all 7 policies are attached to the role.
4. **Wait longer** — First metric pull can take up to 15 minutes.

---

### Issue: Datadog Agent not appearing in Infrastructure

1. In EC2 Instance Connect, run: `sudo systemctl status datadog-agent`
2. If stopped: `sudo systemctl start datadog-agent`
3. Check agent status: `sudo datadog-agent status`
4. Verify the API key: `sudo cat /etc/datadog-agent/datadog.yaml | grep api_key`

---

### Issue: Logs not appearing in Datadog

1. Verify the Forwarder Lambda stack is `CREATE_COMPLETE` in CloudFormation.
2. Check the Lambda function's CloudWatch logs for errors:
   - Go to **Lambda → datadog-forwarder → Monitor → View CloudWatch logs**
3. Verify the subscription filter is active on the log group.
4. Ensure the API key in the CloudFormation stack is correct.

---

### Issue: RDS metrics not appearing

1. Wait 10–15 minutes after enabling Enhanced Monitoring.
2. Verify `AmazonRDSReadOnlyAccess` is attached to `DatadogIntegrationRole`.
3. In Datadog AWS Integration, confirm `RDS` is toggled on in **Metric Collection**.

---

### Issue: CloudFormation stack fails

1. Click on the failed stack → **Events** tab → Find the first `FAILED` event.
2. Common cause: API key incorrect. Delete the stack and recreate.
3. Check IAM permissions — your AWS user needs `CloudFormationFullAccess`.

---

## Lab Teardown

> ⚠️ **Important:** Complete these steps to avoid ongoing AWS charges.

### AWS Resources to Delete

| Resource | Console Path | Action |
|----------|-------------|--------|
| EC2 Instance | EC2 → Instances | Select → Instance State → Terminate |
| RDS Instance | RDS → Databases | Select → Actions → Delete (skip final snapshot) |
| S3 Bucket | S3 | Empty bucket → Delete bucket |
| Lambda Function | Lambda → Functions | Delete `workshop-demo-function` |
| CloudFormation Stack | CloudFormation | Select `datadog-forwarder` → Delete |
| IAM Role | IAM → Roles | Delete `DatadogIntegrationRole` |
| Key Pair | EC2 → Key Pairs | Delete `workshop-keypair` |
| Security Groups | EC2 → Security Groups | Delete workshop security groups |

### Datadog Cleanup

1. **Integrations → Amazon Web Services** → Delete the AWS account.
2. **Monitors → Manage Monitors** → Delete all workshop monitors.
3. **Dashboards** → Delete `AWS Workshop — Operations Overview`.
4. **Organization Settings → API Keys** → Revoke `workshop-key`.

---

## Summary

Congratulations! You have completed the **AWS × Datadog Integration Workshop**. Here is what you accomplished:

| Module | Skill Gained |
|--------|-------------|
| IAM Integration | Secure cross-account role setup with External ID |
| Service Metrics | CloudWatch metric ingestion for 8+ AWS services |
| Agent Install | Host-level monitoring via Datadog Agent |
| Log Forwarding | CloudWatch → Datadog log pipeline |
| Monitors | Threshold and log-based alerting |
| Dashboards | Multi-panel operational dashboard |
| RDS | Database performance monitoring |
| Lambda | Serverless function error tracking |

### Next Steps

- Explore **APM (Application Performance Monitoring)** — instrument your app code
- Set up **Synthetic Monitoring** — create uptime checks for your EC2 HTTP endpoint
- Configure **PagerDuty or Slack notification channels** in Monitor settings
- Explore **Datadog Security Monitoring** for CloudTrail event analysis
- Try **Watchdog** — Datadog's AI anomaly detection feature

---

*Workshop Version 1.0 · AWS × Datadog Integration · UI-Based Lab*  
*All steps performed via AWS Management Console and Datadog Web UI — no CLI or automation required*
