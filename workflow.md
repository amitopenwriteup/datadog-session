# ⚡ Datadog Automation Workshop
### Complete Guide to Workflow Automation, App Builder, Agent Builder & Forms

---

## Table of Contents

1. [Introduction to Datadog Automation](#1-introduction-to-datadog-automation)
2. [Automation Navigation Overview](#2-automation-navigation-overview)
3. [Workflow Automation](#3-workflow-automation)
4. [Action Catalog](#4-action-catalog)
5. [Building Your First Workflow](#5-building-your-first-workflow)
6. [Common Workflow Patterns](#6-common-workflow-patterns)
7. [App Builder](#7-app-builder)
8. [Building Your First App](#8-building-your-first-app)
9. [Agent Builder (Preview)](#9-agent-builder-preview)
10. [Forms (Preview)](#10-forms-preview)
11. [Case Management via Automation](#11-case-management-via-automation)
12. [Best Practices](#12-best-practices)
13. [Hands-On Labs](#13-hands-on-labs)

---

## 1. Introduction to Datadog Automation

Datadog **Automation** is a no-code/low-code platform for building operational workflows, internal tools, and AI agents — all connected to your observability data and external services.

### Why Automation Matters

| Manual Process | Automated with Datadog |
|----------------|------------------------|
| Engineer manually restarts a pod after alert | Workflow auto-restarts on alert trigger |
| Ops team clicks through UI to scale infra | One-click App Builder button triggers scaling |
| Copy-paste incident data into Jira | Workflow auto-creates Jira ticket on incident declare |
| ChatOps via ad-hoc Slack messages | Structured Forms capture inputs, trigger workflows |
| On-call engineer runs same runbook every time | Agent Builder runs the runbook autonomously |

### Automation Components

```
Automation
├── Workflow Automation   ← Trigger-based automated action chains
│   ├── New Workflow      ← Create from scratch
│   └── Action Catalog    ← Pre-built action blocks
│
├── App Builder           ← Build internal tools & dashboards
│   └── New App           ← Create a custom UI app
│
├── Agent Builder [PREVIEW] ← AI agents for autonomous tasks
│
├── Forms [PREVIEW]       ← Structured input collection
│
└── Case Management       ← Track automation-generated work items
```

---

## 2. Automation Navigation Overview

### Navigate to: Automation (Sidebar)

```
Automation
├── Actions
│   ├── Workflow Automation  → View/manage all workflows
│   ├── Action Catalog       → Browse available action blocks
│   └── New Workflow         → Start a new workflow
│
├── App Builder
│   ├── (App list)           → All created apps
│   └── New App              → Create a new internal app
│
├── Case Management          → Work items linked to automations
│
├── Agent Builder  [PREVIEW] → AI-powered autonomous agents
│
└── Forms          [PREVIEW] → Input forms that trigger workflows
```

---

## 3. Workflow Automation

### What is a Workflow?

A **Workflow** is an automated sequence of actions triggered by an event — a monitor alert, an incident declaration, a schedule, or a manual action.

```
Trigger → Steps → Actions → Notifications → End
```

### Workflow Anatomy

```
┌─────────────────────────────────────────────────────┐
│                    WORKFLOW                          │
│                                                      │
│  [Trigger]                                           │
│     ↓                                                │
│  [Step 1: Condition check]                           │
│     ↓ YES              ↓ NO                          │
│  [Step 2: Action A]  [Step 3: Action B]              │
│     ↓                                                │
│  [Step 4: Notification]                              │
│     ↓                                                │
│  [End]                                               │
└─────────────────────────────────────────────────────┘
```

### Trigger Types

| Trigger | Description | Use Case |
|---------|-------------|----------|
| **Monitor Alert** | Fires when a Datadog monitor enters ALERT state | Auto-remediation |
| **Incident Declared** | Fires when an incident is created | Stakeholder notifications |
| **Schedule** | Runs on a cron schedule | Daily reports, cleanup jobs |
| **Manual** | Triggered by a human via UI or API | On-demand runbooks |
| **Webhook** | External HTTP call triggers workflow | CI/CD integration |
| **Security Signal** | Fires on a security finding | Auto-quarantine |
| **Case Event** | Fires on case creation/update | Project management sync |

### Workflow States

```
Draft       → Being built, not yet active
Active      → Running and responding to triggers
Paused      → Temporarily disabled
Archived    → Retired workflow, kept for history
```

---

## 4. Action Catalog

### Navigate to: Automation → Actions → Action Catalog

The Action Catalog is a library of **pre-built action blocks** you can drop into any workflow.

### Action Categories

#### Infrastructure
```
AWS Actions:
  ├── EC2: Start / Stop / Reboot instance
  ├── EC2: Describe instances
  ├── ECS: Update service (scale in/out)
  ├── Lambda: Invoke function
  ├── RDS: Create snapshot / Reboot instance
  ├── S3: Get / Put object
  └── Auto Scaling: Set desired capacity

Kubernetes Actions:
  ├── Get pod / deployment / node details
  ├── Scale deployment
  ├── Restart deployment (rollout restart)
  ├── Cordon / Uncordon node
  ├── Delete pod (force restart)
  └── Get pod logs
```

#### Notifications & Communication
```
Slack:
  ├── Send message to channel
  ├── Send direct message
  └── Create Slack thread

Email:
  └── Send email to user/group

PagerDuty:
  ├── Trigger incident
  ├── Acknowledge incident
  └── Resolve incident

Microsoft Teams:
  └── Send message to channel
```

#### Ticketing & Project Management
```
Jira:
  ├── Create issue
  ├── Update issue
  ├── Add comment
  └── Transition issue status

ServiceNow:
  ├── Create incident
  ├── Update record
  └── Close incident

GitHub:
  ├── Create issue
  ├── Create pull request
  └── Add comment
```

#### Datadog Native Actions
```
Datadog:
  ├── Create incident
  ├── Update incident
  ├── Mute monitor
  ├── Create downtime
  ├── Create case
  ├── Post to Event Stream
  └── Run synthetic test
```

#### HTTP & Custom
```
HTTP Request:
  ├── GET / POST / PUT / DELETE
  ├── Custom headers and auth
  └── Parse JSON response

JavaScript:
  └── Run custom JS logic / transformations

Data Transform:
  ├── Filter arrays
  ├── Map values
  └── Format strings
```

---

## 5. Building Your First Workflow

### Navigate to: Automation → New Workflow

### Step-by-Step: Auto-Remediation Workflow

**Scenario:** When a monitor detects high CPU on a host, automatically restart the affected service and notify the team.

---

#### Step 1 — Name & Configure the Workflow

```
Name:        Auto-Restart on High CPU
Description: Restarts the affected service when CPU exceeds 90%
             and notifies #ops-alerts Slack channel
Tags:        team:platform, env:prod
```

#### Step 2 — Set the Trigger

```
Trigger type: Monitor Alert
Monitor:      [Infrastructure] High CPU - Production
State:        Alert (only trigger on ALERT, not WARN)
```

#### Step 3 — Add a Condition (Branching Logic)

```
Condition: Is environment = "production"?
  YES → Continue to restart action
  NO  → Skip restart, only notify
```

#### Step 4 — Add Action: Restart Kubernetes Deployment

```
Action type:  Kubernetes → Rollout Restart Deployment
Namespace:    {{ trigger.monitor.tags.namespace }}
Deployment:   {{ trigger.monitor.tags.service }}
Cluster:      prod-cluster-us-east-1
```

#### Step 5 — Add Action: Send Slack Notification

```
Action type:  Slack → Send message to channel
Channel:      #ops-alerts
Message:
  🔄 *Auto-remediation triggered*
  Monitor: {{ trigger.monitor.name }}
  Host: {{ trigger.monitor.tags.host }}
  Action: Restarted deployment {{ trigger.monitor.tags.service }}
  Time: {{ workflow.startedAt }}
  Status: Awaiting recovery...
```

#### Step 6 — Add Action: Wait & Verify

```
Action type:  Wait
Duration:     120 seconds
Reason:       Allow service to restart and stabilize
```

#### Step 7 — Add Action: Check Monitor Status

```
Action type:  Datadog → Get Monitor Status
Monitor ID:   {{ trigger.monitor.id }}
```

#### Step 8 — Final Branch: Success or Escalate

```
Condition: Is monitor status = "OK"?
  YES → Send Slack: "✅ Service recovered successfully"
  NO  → Create Datadog Incident (SEV-2) + Page on-call team
```

#### Step 9 — Activate Workflow

```
Review → Save → Toggle to "Active"
```

---

### Workflow Variables

Use `{{ }}` syntax to reference dynamic values:

```
Trigger variables:
  {{ trigger.monitor.name }}         ← Monitor name
  {{ trigger.monitor.id }}           ← Monitor ID
  {{ trigger.monitor.tags.host }}    ← Tag value
  {{ trigger.monitor.tags.service }} ← Service tag
  {{ trigger.monitor.message }}      ← Alert message

Step output variables:
  {{ steps.step_name.output.field }} ← Output from a previous step

System variables:
  {{ workflow.id }}                  ← Workflow run ID
  {{ workflow.startedAt }}           ← Start timestamp
  {{ workflow.triggeredBy }}         ← Who/what triggered it

Input parameters (for manual workflows):
  {{ inputs.environment }}
  {{ inputs.reason }}
```

---

## 6. Common Workflow Patterns

### Pattern 1 — Incident Auto-Notification

```
Trigger: Incident declared (SEV-1 or SEV-2)
Actions:
  1. Post to #incidents Slack channel with incident details
  2. Create Jira ticket linked to incident
  3. Update status page: "Investigating issue"
  4. Schedule 15-minute reminder for status update
```

### Pattern 2 — Auto-Scale on High Load

```
Trigger: Monitor → High request queue depth
Condition: Current replica count < max replicas
Actions:
  1. AWS ECS → Update service desired count (+2)
  2. Slack: "Scaled up checkout-api from N to N+2 replicas"
  3. Datadog: Post event "Auto-scaled checkout-api"
  4. Wait 5 minutes
  5. Check if queue depth normalized
  6. If YES: Slack "✅ Queue depth normalized"
  7. If NO: Page on-call team
```

### Pattern 3 — Security Signal Response

```
Trigger: Security signal → Unauthorized API access (HIGH severity)
Actions:
  1. HTTP Request → Block IP via WAF API
  2. Datadog: Create security incident
  3. Slack #security-alerts: "🔒 IP blocked: {{ signal.attributes.ip }}"
  4. ServiceNow: Create security ticket
  5. Email: Notify CISO
```

### Pattern 4 — Daily Cost Report

```
Trigger: Schedule → Every weekday at 9:00 AM
Actions:
  1. HTTP Request → Datadog Cost API (get yesterday's spend)
  2. JavaScript → Format cost breakdown by team
  3. Slack #finops-daily:
     "📊 Yesterday's AWS spend: $X,XXX
      Top spender: {{ team }} ($XXX)
      Budget status: {{ pct }}% of monthly budget used"
```

### Pattern 5 — Deployment Verification

```
Trigger: Webhook (from CI/CD pipeline on deploy)
Inputs:  service, version, environment
Actions:
  1. Wait 3 minutes (allow rollout)
  2. Datadog: Run synthetic test for the service
  3. Condition: Did synthetic test pass?
     YES: Slack "✅ {{ inputs.service }} v{{ inputs.version }} deployed successfully"
     NO:  Trigger rollback workflow + Page on-call
```

---

## 7. App Builder

### Navigate to: Automation → App Builder

### What is App Builder?

App Builder lets you create **custom internal tools** — web UIs with buttons, forms, tables, and dropdowns — that trigger Datadog actions or external APIs. No frontend development required.

### Use Cases

```
Operations Dashboard:
  - View all unhealthy pods → click to restart
  - See open incidents → click to acknowledge
  - Scale services up/down with a slider

FinOps Tool:
  - Show AWS cost by team
  - Set budget alerts with a form
  - Approve/deny cost overruns

Security Console:
  - Review open findings
  - One-click block IP
  - Assign findings to team members

Release Manager:
  - View deployments in progress
  - Trigger rollback with reason field
  - Approve promotion to production
```

### App Builder Components

#### UI Components

| Component | Description |
|-----------|-------------|
| **Table** | Display data from a query or API |
| **Button** | Trigger an action or workflow |
| **Form** | Collect user inputs |
| **Text Input** | Single-line text field |
| **Dropdown** | Select from a list of options |
| **Date Picker** | Select a date/time |
| **Toggle** | Boolean on/off switch |
| **Text Block** | Static or dynamic display text |
| **Metric Value** | Show a single KPI number |
| **Chart** | Embed a metrics graph |
| **JSON Viewer** | Display structured data |

#### Data Sources

```
Datadog:
  ├── Metrics query
  ├── Logs query
  ├── Monitor list
  ├── Incidents
  └── Kubernetes resources

External:
  ├── HTTP API (any REST endpoint)
  ├── AWS services
  └── Custom integrations
```

---

## 8. Building Your First App

### Navigate to: Automation → App Builder → New App

### Step-by-Step: Kubernetes Pod Manager App

**Objective:** Build an app that shows unhealthy pods and lets engineers restart them with one click.

---

#### Step 1 — Name Your App

```
Name:        Kubernetes Pod Manager
Description: View unhealthy pods and restart them from one place
Icon:        ⚙️
```

#### Step 2 — Add a Data Query

```
Query name:  unhealthy_pods
Type:        Kubernetes
Action:      List Pods
Cluster:     prod-cluster-us-east-1
Namespace:   All
Filter:      status != "Running"
```

#### Step 3 — Add a Table Component

```
Component:   Table
Data source: {{ queries.unhealthy_pods.output }}
Columns:
  - Namespace
  - Pod Name
  - Status
  - Restarts
  - Node
  - Age
```

#### Step 4 — Add a Restart Button

```
Component:   Button
Label:       🔄 Restart Selected Pod
Style:       Warning (orange)
On click:    Run action → Kubernetes: Delete Pod
             Namespace: {{ table.selectedRow.namespace }}
             Pod name:  {{ table.selectedRow.name }}
Confirm:     "Are you sure you want to restart {{ table.selectedRow.name }}?"
```

#### Step 5 — Add a Success Notification

```
After action:  Show toast notification
Message:       "Pod {{ table.selectedRow.name }} restart initiated"
Type:          Success (green)
```

#### Step 6 — Add a Filter Input

```
Component:     Dropdown
Label:         Filter by Namespace
Options:       {{ queries.get_namespaces.output }}
On change:     Refresh unhealthy_pods query with selected namespace
```

#### Step 7 — Publish the App

```
Click "Publish" → Share URL with team
Access control:
  ○ Anyone in the org
  ● Only specific teams
  ○ Only specific users
```

---

### App Layout Tips

```
Good layout structure:
┌─────────────────────────────────────────────┐
│  Title: Kubernetes Pod Manager               │
│  [Namespace filter ▼]  [Refresh button]     │
├─────────────────────────────────────────────┤
│  Table: Unhealthy Pods                       │
│  Namespace │ Pod Name │ Status │ Restarts    │
│  ──────────────────────────────────────────  │
│  default   │ api-5f4  │ Error  │ 12          │
│  payments  │ db-7c2   │ Crash  │  4          │
├─────────────────────────────────────────────┤
│  [🔄 Restart Selected]  [📋 View Logs]       │
└─────────────────────────────────────────────┘
```

---

## 9. Agent Builder (Preview)

### Navigate to: Automation → Agent Builder

### What is Agent Builder?

**Agent Builder** lets you create **AI-powered autonomous agents** that can reason, make decisions, and take multi-step actions — like a smart runbook that executes itself.

### How Agents Differ from Workflows

| | Workflow | Agent |
|-|----------|-------|
| **Decision making** | Fixed branching logic (if/else) | AI reasoning — adapts to context |
| **Steps** | Predefined sequence | Dynamic — agent decides next step |
| **Handles surprises** | Falls to error path | Tries alternative approaches |
| **Best for** | Predictable, repetitive tasks | Complex, variable investigations |

### Agent Capabilities

```
An agent can:
  ✅ Query Datadog metrics, logs, and traces
  ✅ Read monitor states and alert history
  ✅ Execute Kubernetes operations
  ✅ Call external APIs
  ✅ Create incidents, cases, and tickets
  ✅ Send Slack messages
  ✅ Reason about data and summarize findings
  ✅ Chain multiple tool calls based on findings
```

### Creating an Agent

**Step 1 — Define the Agent's Goal**
```
Name:        On-Call First Responder
Goal:        When paged about a service issue, investigate 
             the root cause and provide a summary with
             recommended actions before a human steps in.
```

**Step 2 — Give the Agent Tools**
```
Available tools:
  ✅ Query Datadog logs (last 30 minutes)
  ✅ Get metric values for a service
  ✅ Check monitor states
  ✅ List recent deployments (via webhook)
  ✅ Get Kubernetes pod status
  ✅ Send Slack message
  ✅ Create Datadog incident
```

**Step 3 — Define the Instructions**
```
System prompt:
  You are an on-call assistant for the Platform SRE team.
  When triggered, you will:
  1. Identify the affected service from the alert context
  2. Check recent error logs for that service (last 30 min)
  3. Check if there were any recent deployments
  4. Check Kubernetes pod health for that service
  5. Look for any correlated metric anomalies
  6. Summarize findings and suggest 2-3 remediation steps
  7. Post the summary to #ops-alerts Slack channel
  8. If severity appears SEV-1 or SEV-2, declare an incident

  Be concise. Format your summary as:
  - What's broken
  - Probable cause
  - Recommended actions
```

**Step 4 — Set the Trigger**
```
Trigger: Monitor alert → Any P1/P2 monitor
OR
Trigger: Manual → "Investigate this alert" button in App Builder
```

### Example Agent Output

```
🤖 Agent First Responder - Investigation Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Service: checkout-api | Time: 14:32 UTC

🔴 WHAT'S BROKEN
  Error rate spiked to 18.4% (threshold: 5%)
  ~1,200 failed requests/min in the last 10 minutes

🔍 PROBABLE CAUSE
  Deployment checkout-api v2.5.1 was pushed 14 minutes ago.
  Error logs show: "database connection timeout" repeated 847x.
  DB connection pool exhausted: 100/100 connections used.
  No pod restarts detected — issue is connection saturation.

✅ RECOMMENDED ACTIONS
  1. Immediate: Rollback to v2.5.0
     kubectl rollout undo deployment/checkout-api
  2. Short-term: Increase DB connection pool limit (currently 100)
  3. Investigate: Review DB query optimization in v2.5.1 diff
  
📎 Runbook: https://wiki.internal/runbooks/checkout-errors
```

---

## 10. Forms (Preview)

### Navigate to: Automation → Forms

### What are Forms?

**Forms** are structured input pages that capture data from engineers or operators and feed it into a workflow — ensuring consistent, validated inputs for automation.

### When to Use Forms

```
Use forms when a workflow needs human input before executing:
  - Reason for a manual action (required audit trail)
  - Target environment selection (dev / staging / prod)
  - Approval confirmation before destructive actions
  - Incident declaration with structured fields
  - Change request submission
```

### Form Components

| Field Type | Description |
|------------|-------------|
| **Text Input** | Short text (name, reason) |
| **Textarea** | Long text (description, notes) |
| **Dropdown** | Select from predefined options |
| **Multi-select** | Choose multiple options |
| **Checkbox** | Boolean yes/no |
| **Date/Time Picker** | Schedule selection |
| **Number Input** | Numeric values (replica count, timeout) |
| **File Upload** | Attach supporting documents |

### Building a Form: Deployment Rollback Request

```
Form name: Deployment Rollback Request
Purpose:   Capture rollback details before executing

Fields:
  1. Service name (dropdown — list from Datadog)
     Label: "Which service needs rollback?"
     Required: Yes

  2. Target version (text input)
     Label: "Roll back to which version?"
     Placeholder: "e.g., v2.5.0"
     Required: Yes

  3. Reason (dropdown)
     Label: "Reason for rollback"
     Options:
       - Elevated error rate
       - Performance regression
       - Failed health checks
       - Customer complaints
       - Security vulnerability
     Required: Yes

  4. Additional context (textarea)
     Label: "Any additional notes?"
     Required: No

  5. Confirmation (checkbox)
     Label: "I confirm this rollback is necessary and approved"
     Required: Yes

On Submit → Trigger workflow: Execute Rollback
            Pass all form values as workflow inputs
```

### Form Access & Sharing

```
Sharing options:
  Direct URL:  share.datadoghq.com/forms/rollback-request
  Embedded:    Add to App Builder app as a modal
  Slack:       /dd-form rollback (via Slack bot)
  Monitor:     Include form link in alert notification
```

---

## 11. Case Management via Automation

### Navigate to: Automation → Case Management

### Auto-Creating Cases from Workflows

Workflows can automatically create Cases to track follow-up:

```yaml
# At the end of an incident workflow:
Action: Datadog → Create Case
  Title:    Post-incident: {{ trigger.incident.title }}
  Type:     Investigation
  Priority: P{{ trigger.incident.severity_number }}
  Assignee: {{ trigger.incident.commander }}
  Tags:     service:{{ trigger.incident.service }}, 
            env:prod
  Description: |
    Incident {{ trigger.incident.id }} has been resolved.
    Root cause investigation required.
    
    Incident duration: {{ incident.duration }}
    Affected service: {{ incident.service }}
    Responders: {{ incident.responders }}
```

### Case Lifecycle in Automation

```
Workflow creates Case → Case assigned to engineer
        ↓
Engineer works the case → Updates status to "In Progress"
        ↓
Engineer links PRs, runbook updates, config changes
        ↓
Case closed → Metrics captured (resolution time, etc.)
        ↓
Analytics → Feed back into post-mortem process
```

### Bulk Case Operations via Workflow

```
Use case: Weekly cleanup workflow

Trigger: Schedule → Every Monday 9 AM
Actions:
  1. Query: Get all Cases older than 14 days, status=Open
  2. For each case:
     - If no activity in 7 days → Add comment "Auto-reminder: This case needs attention"
     - If no activity in 14 days → Escalate to team lead
  3. Slack summary: "Weekly Case Review: X cases need attention"
```

---

## 12. Best Practices

### Workflow Design

```
✅ Start simple — build MVP workflow, then add complexity
✅ Always add a notification action — visibility is critical
✅ Test in non-production first — use dev/staging monitors as triggers
✅ Add error handling — what happens if an action fails?
✅ Use conditions before destructive actions (restart, scale, delete)
✅ Add human approval step for high-risk actions
✅ Log every action to the Datadog Event Stream for audit trail
✅ Set timeouts on wait steps — don't leave workflows hanging
✅ Name steps descriptively — "restart-pod" not "step-3"
```

### App Builder

```
✅ Add confirmation dialogs before destructive buttons
✅ Show current state in the table before offering actions
✅ Use role-based access — not everyone should see all apps
✅ Keep apps focused — one app per use case, not a mega-app
✅ Add loading indicators for actions that take time
✅ Test with realistic data before sharing with the team
```

### Security

```
✅ Scope workflow permissions to least privilege
✅ Require "reason" input for any manual destructive action
✅ Never hardcode secrets — use Datadog's secret store
✅ Audit workflow run history monthly
✅ Disable unused workflows (don't delete — archive them)
✅ Review who has access to run each app
```

### Agent Builder

```
✅ Define clear, specific goals — vague prompts produce vague results
✅ Limit tool access to what the agent actually needs
✅ Always include a Slack notification as a final step
✅ Test agents with realistic scenarios before production use
✅ Set a maximum steps limit to prevent runaway agents
✅ Review agent outputs weekly during preview phase
```

---

## 13. Hands-On Labs

### Lab 1 — Build a Monitor Alert Workflow (25 min)

**Objective:** Create a workflow that sends a Slack message and creates a Datadog case when any monitor alerts.

```
Steps:
1. Navigate: Automation → New Workflow
2. Name: Lab - Alert to Slack & Case
3. Trigger: Monitor Alert
   - Select any monitor in your environment
   - State: Alert

4. Add Step 1 — Slack notification:
   Action: Slack → Send message
   Channel: (your Slack channel, or use email if no Slack)
   Message:
     🚨 Monitor Alert!
     Monitor: {{ trigger.monitor.name }}
     Status: {{ trigger.monitor.status }}
     Time: {{ workflow.startedAt }}

5. Add Step 2 — Create a Case:
   Action: Datadog → Create Case
   Title: Alert: {{ trigger.monitor.name }}
   Type: Investigation
   Priority: P3
   Description: Auto-created from workflow lab

6. Save → Activate workflow
7. Manually trigger your monitor (or use "Test" feature)
8. Verify: Slack message received + Case created
```

---

### Lab 2 — Build a Scheduled Daily Report Workflow (20 min)

**Objective:** Create a workflow that runs every morning and posts a summary.

```
Steps:
1. Navigate: Automation → New Workflow
2. Name: Lab - Daily Morning Summary
3. Trigger: Schedule
   Cron: 0 9 * * 1-5  (9 AM, Monday–Friday)
   Timezone: UTC+05:30

4. Add Step 1 — Get triggered monitors:
   Action: Datadog → List Monitors
   Filter: status:Alert

5. Add Step 2 — Slack message:
   Action: Slack → Send message
   Message:
     📊 Good morning! Daily Datadog Summary
     ─────────────────────────────
     🔴 Triggered monitors: {{ steps.list_monitors.output.count }}
     ⏰ Time: {{ workflow.startedAt }}
     
     Review: https://app.datadoghq.com/monitors/triggered

6. Save → Activate
7. Use "Test" button to verify the workflow runs
```

---

### Lab 3 — Build a Pod Restart App (30 min)

**Objective:** Create a simple App Builder tool to restart Kubernetes pods.

```
Steps:
1. Navigate: Automation → App Builder → New App
2. Name: Lab - Pod Restart Tool

3. Add a Query:
   Name: get_pods
   Type: Kubernetes → List Pods
   Cluster: (your cluster)
   Namespace: default

4. Add a Table component:
   Data: {{ queries.get_pods.output }}
   Columns: name, status, restarts, namespace

5. Add a Button:
   Label: Restart Pod
   On click: Kubernetes → Delete Pod
   Pod: {{ table.selectedRow.name }}
   Namespace: {{ table.selectedRow.namespace }}
   Confirmation: "Restart {{ table.selectedRow.name }}?"

6. Add a Text Block:
   Content: "Select a pod from the table, then click Restart Pod"
   Style: Info banner

7. Preview → Test with a non-production pod
8. Publish app
```

---

### Lab 4 — Create a Form for Incident Declaration (20 min)

**Objective:** Build a structured form to standardize how incidents are declared.

```
Steps:
1. Navigate: Automation → Forms → New Form
2. Name: Declare Incident Form

3. Add fields:
   Field 1: Dropdown
     Label: Affected service
     Options: checkout, payments, auth, api-gateway, database
     Required: Yes

   Field 2: Dropdown
     Label: Severity
     Options: SEV-1, SEV-2, SEV-3, SEV-4
     Required: Yes

   Field 3: Textarea
     Label: Describe the issue
     Placeholder: What is broken? What is the user impact?
     Required: Yes

   Field 4: Dropdown
     Label: Detection method
     Options: Monitor alert, Manual observation, Customer report, Synthetic test
     Required: Yes

   Field 5: Checkbox
     Label: "I confirm this is a real incident, not a test"
     Required: Yes

4. On Submit → Trigger workflow: "Declare Incident"
   (or Datadog → Create Incident action directly)

5. Copy form URL → Share with your team
6. Submit a test form entry
7. Verify: Incident created in Incident Management
```

---

### Lab 5 — Explore the Action Catalog (15 min)

**Objective:** Familiarize yourself with available automation building blocks.

```
Steps:
1. Navigate: Automation → Actions → Action Catalog
2. Browse the following categories and note 2 actions each:
   - Kubernetes
   - AWS
   - Slack
   - Jira (or GitHub)
   - Datadog native

3. For each action you find interesting, note:
   - Action name
   - Required inputs
   - Output variables available

4. Create a new workflow (Draft, don't activate)
5. Add 3 different action types from the catalog
6. Configure each with placeholder values
7. Review the step dependencies and output variables

Reflection questions:
  - Which action would you use to auto-scale your service?
  - Which action would page your on-call team?
  - Which action would create a Jira ticket from an incident?
```

---

## Appendix

### Workflow Cron Reference

```
Every 5 minutes:         */5 * * * *
Every hour:              0 * * * *
Daily at 9 AM:           0 9 * * *
Weekdays at 9 AM:        0 9 * * 1-5
Every Monday at 8 AM:    0 8 * * 1
First of month at noon:  0 12 1 * *
```

### Common Variable Reference

```
Monitor trigger:
  {{ trigger.monitor.name }}
  {{ trigger.monitor.id }}
  {{ trigger.monitor.status }}
  {{ trigger.monitor.tags }}
  {{ trigger.monitor.message }}

Incident trigger:
  {{ trigger.incident.id }}
  {{ trigger.incident.title }}
  {{ trigger.incident.severity }}
  {{ trigger.incident.commander }}

System:
  {{ workflow.id }}
  {{ workflow.startedAt }}
  {{ workflow.triggeredBy.name }}

Step outputs:
  {{ steps.STEP_NAME.output }}
  {{ steps.STEP_NAME.output.FIELD }}
```

### Automation Use Case Matrix

| Problem | Solution |
|---------|----------|
| Monitor fires, nobody sees it | Workflow → Slack notification |
| Repetitive restart tasks | App Builder → Restart button |
| On-call doesn't know where to start | Agent Builder → Auto-investigate |
| Inconsistent incident declarations | Forms → Structured declaration |
| Post-incident action items forgotten | Workflow → Auto-create Cases |
| Daily cost reports are manual | Scheduled workflow → Daily report |
| Deployments need verification | Webhook trigger → Run synthetic test |
| Security signals need fast response | Security trigger → Block IP workflow |

---

### Workshop Resources

- 📘 Workflow Automation Docs: https://docs.datadoghq.com/service_management/workflows/
- 🛠️ App Builder Docs: https://docs.datadoghq.com/service_management/app_builder/
- 🤖 Agent Builder Docs: https://docs.datadoghq.com/service_management/agent_builder/
- 📋 Action Catalog: https://app.datadoghq.com/workflow/action-catalog
- 🎓 Datadog Learning Center: https://learn.datadoghq.com
- 💬 Automation Community: https://dtdg.co/community

---

*Workshop Version 1.0 | Datadog Automation Training Guide*
