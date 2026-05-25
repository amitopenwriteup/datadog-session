Here's the complete, detailed guide to integrate Datadog with AWS:

---

# 🔗 Datadog + AWS Integration — Complete Step-by-Step Guide

---

## Part 1: Prerequisites

Before starting, ensure you have:

- ✅ An **AWS account** with admin or IAM-capable access
- ✅ A **Datadog account** (your current org: `owtest`)
- ✅ Your **Datadog API Key** — find it at [API Keys](/organization-settings/api-keys)
- ✅ Your AWS IAM user must have these permissions to run CloudFormation:
  - `cloudformation:*`
  - `iam:CreateRole`, `iam:PutRolePolicy`, `iam:AttachRolePolicy`
  - `lambda:CreateFunction`, `lambda:AddPermission` (if enabling log forwarding)
  - `secretsmanager:*` (for storing keys)

---

## Part 2: Core AWS Integration (CloudFormation — Recommended)

### Step 1: Open the AWS Integration Page
1. In Datadog, go to [Integrations → Amazon Web Services](/integrations/amazon-web-services)
2. Click **Add AWS Account(s)**

### Step 2: Select Setup Method
1. Choose **Automatically using CloudFormation** (pre-selected)
2. This is the fastest and most reliable method

### Step 3: Configure Integration Settings
1. **Select AWS Regions** — check all regions where you run AWS resources (e.g., `us-east-1`, `ap-south-1`)
2. **Datadog API Key** — select your existing key from the dropdown, or create a new one
3. **Send Logs to Datadog** — toggle **Yes** if you want AWS service logs (deploys the Datadog Forwarder Lambda function)
4. **Cloud Security** — toggle **Yes** to enable Cloud Security Posture Management (optional, scans for misconfigurations)

### Step 4: Launch CloudFormation in AWS
1. Click **Launch CloudFormation Template**
2. This opens the **AWS Console** in a new tab with a pre-filled CloudFormation stack
3. All parameters are already filled — **do not change them** unless you have specific requirements

### Step 5: Create the Stack in AWS Console
1. On the AWS CloudFormation page, scroll to the bottom
2. Check **all three acknowledgment boxes**:
   - ☑️ I acknowledge that AWS CloudFormation might create IAM resources
   - ☑️ I acknowledge that AWS CloudFormation might create IAM resources with custom names
   - ☑️ I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND
3. Click **Create stack**
4. Wait **3–5 minutes** for all stacks to complete (you'll see status: `CREATE_COMPLETE`)

**What this creates in your AWS account:**
| Resource | Purpose |
|----------|---------|
| IAM Role (`DatadogIntegrationRole`) | Allows Datadog to read your AWS data |
| IAM Policy | Read-only access to EC2, RDS, S3, CloudWatch, Lambda, etc. |
| Datadog Forwarder Lambda | Forwards logs from CloudWatch/S3 to Datadog (if enabled) |
| Secrets Manager secret | Stores your Datadog API/App keys |

### Step 6: Confirm in Datadog
1. Go back to the Datadog browser tab
2. Click **Ready!**
3. Wait **\~10 minutes** for data to appear

### Step 7: Verify Data is Flowing
1. Go to [Infrastructure List](/infrastructure) — your EC2 hosts should appear with AWS tags
2. Check the [AWS Overview dashboard](/dash/integration/7/aws-overview) for CloudWatch metrics
3. Search `aws.*` in [Metric Explorer](https://app.datadoghq.com/metric/explorer) to confirm metrics are flowing

---

## Part 3: Enable Individual AWS Service Integrations

After the core integration is active, enable specific AWS services:

### Step 8: Configure Metric Collection
1. Go to [AWS Integration page](/integrations/amazon-web-services)
2. Click your AWS account → **Metric Collection** tab
3. Enable the services you use:

| Service | Toggle | What You Get |
|---------|--------|-------------|
| **EC2** | ✅ On | Instance metrics, status checks, tags |
| **RDS** | ✅ On | Database CPU, connections, read/write latency |
| **S3** | ✅ On | Bucket size, request count, errors |
| **Lambda** | ✅ On | Invocations, duration, errors, throttles |
| **ELB/ALB** | ✅ On | Request count, latency, HTTP errors |
| **ECS** | ✅ On | Container metrics, task counts |
| **ElastiCache** | ✅ On | Redis/Memcached metrics |
| **SQS** | ✅ On | Queue depth, messages sent/received |
| **SNS** | ✅ On | Messages published, delivery failures |
| **DynamoDB** | ✅ On | Read/write capacity, throttle events |
| **CloudFront** | ✅ On | CDN request count, error rate |
| **Route 53** | ✅ On | DNS query count, health checks |

4. Click **Save**

> 💡 Many services are auto-enabled when Datadog detects data from your account.

---

## Part 4: Install the Datadog Agent on EC2 Instances

CloudWatch metrics are polled every 10 minutes. For **real-time host-level visibility** (CPU, memory, disk, network, processes), install the Agent.

### Step 9: Install the Agent on EC2
1. Go to [Agent Installation page](/fleet/install-agent/latest?platform=overview)
2. Select your OS (Amazon Linux, Ubuntu, etc.)
3. SSH into your EC2 instance and run the one-line install command:

**Amazon Linux / CentOS / RHEL:**
```bash
DD_API_KEY=<YOUR_API_KEY> DD_SITE="us5.datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

**Ubuntu / Debian:**
```bash
DD_API_KEY=<YOUR_API_KEY> DD_SITE="us5.datadoghq.com" bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
```

**Windows (PowerShell as Admin):**
```powershell
$env:DD_API_KEY="<YOUR_API_KEY>"
$env:DD_SITE="us5.datadoghq.com"
. { iwr -useb https://install.datadoghq.com/scripts/install_script_agent7.ps1 } | iex
```

4. Verify the Agent is running:
```bash
sudo datadog-agent status
```

5. Within 2 minutes, the host appears in [Infrastructure List](/infrastructure) with a 🦴 bone icon

### Step 10: Repeat for All Critical Hosts
- Use **AWS Systems Manager (SSM)** Run Command to install across multiple EC2 instances at once
- Or use your configuration management tool (Ansible, Chef, Puppet, Terraform)

---

## Part 5: Set Up Log Forwarding

### Step 11: Configure AWS Log Collection
There are two methods:

**Method A — Amazon Data Firehose (Recommended for high-volume CloudWatch logs)**
1. Follow the [Amazon Data Firehose setup docs](https://docs.datadoghq.com/logs/guide/send-aws-services-logs-with-the-datadog-kinesis-firehose-destination/)

**Method B — Datadog Forwarder Lambda (Required for Lambda traces/custom metrics)**
1. If you enabled "Send Logs to Datadog" in Step 3, the Forwarder Lambda is already deployed
2. Go to [AWS Integration page](/integrations/amazon-web-services) → select your account → **Log Collection** tab
3. Under **Datadog Forwarder Lambda**, enter the ARN of the Lambda function → click **Add**
4. Under **Log Autosubscription**, toggle on the services you want logs from:
   - CloudTrail, VPC Flow Logs, RDS, Lambda, ALB, S3 access logs, etc.
5. Click **Save**

### Step 12: Verify Logs
1. Go to [Logs Explorer](/logs)
2. Filter by `source:aws*` or specific sources like `source:cloudtrail`, `source:s3`
3. Logs should appear within **5 minutes** of configuration

---

## Part 6: Validation Checklist

| Check | Where to Verify | Expected Result |
|-------|----------------|-----------------|
| AWS account connected | [AWS Integration](/integrations/amazon-web-services) | Account listed with green status |
| CloudWatch metrics flowing | [Metric Explorer](https://app.datadoghq.com/metric/explorer) → search `aws.ec2` | Metrics appear with AWS tags |
| EC2 hosts visible | [Infrastructure List](/infrastructure) | Hosts listed with AWS tags (region, instance-type, etc.) |
| Agent installed | [Infrastructure](/infrastructure) → look for 🦴 icon | Agent reporting system metrics |
| Logs arriving | [Logs Explorer](/logs) → `source:aws*` | AWS service logs visible |
| AWS dashboard populated | [AWS Overview](/dash/integration/7/aws-overview) | Widgets showing data |

---

## Part 7: Recommended Next Steps

| Action | Why |
|--------|-----|
| **Tag your AWS resources** | Add `env`, `service`, `team` tags in AWS for unified Datadog tagging |
| **Create monitors** | Set up alerts on key AWS metrics (CPU, disk, errors) |
| **Enable APM on EC2** | Add tracing libraries to your apps for full-stack observability |
| **Set up cost monitoring** | Enable [Cloud Cost Management](https://docs.datadoghq.com/cloud_cost_management/) to track AWS spend |
| **Review security** | Enable [Cloud Security](https://docs.datadoghq.com/security/) for misconfig detection |

---

📚 **Official References:**
- [Getting Started with AWS](https://docs.datadoghq.com/getting_started/integrations/aws/)
- [AWS Manual Setup Guide](https://docs.datadoghq.com/integrations/guide/aws-manual-setup/)
- [AWS Log Forwarding](https://docs.datadoghq.com/logs/guide/send-aws-services-logs-with-the-datadog-lambda-function/)
- [Datadog Agent Installation](https://docs.datadoghq.com/agent/)
