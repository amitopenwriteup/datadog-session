#  Datadog Incident Response Workshop
### Complete Guide to On-Call, Incident Management, Status Pages & Case Management

---

## Table of Contents

1. [Introduction to Incident Response in Datadog](#1-introduction-to-incident-response-in-datadog)
2. [Incident Response Navigation Overview](#2-incident-response-navigation-overview)
3. [On-Call — Teams](#3-on-call--teams)
4. [On-Call — Pages & Escalations](#4-on-call--pages--escalations)
5. [Incident Management](#5-incident-management)
6. [Declaring & Managing an Incident](#6-declaring--managing-an-incident)
7. [Incident Severity & Classification](#7-incident-severity--classification)
8. [Status Pages (NEW)](#8-status-pages-new)
9. [Case Management](#9-case-management)
10. [Incident Workflows & Automation](#10-incident-workflows--automation)
11. [Post-Incident Review](#11-post-incident-review)
12. [Best Practices](#12-best-practices)
13. [Hands-On Labs](#13-hands-on-labs)

---

## 1. Introduction to Incident Response in Datadog

Datadog's **Incident Response** module provides an end-to-end system for detecting, routing, managing, and learning from incidents — all within the same platform used for monitoring and observability.

### Why Unified Incident Response Matters

| Fragmented Tooling | Datadog Incident Response |
|--------------------|---------------------------|
| PagerDuty for alerts + Slack for comms + Jira for tickets | Single unified platform |
| Context lost when switching tools | Full telemetry linked to every incident |
| Manual status page updates | Automated status page publishing |
| Spreadsheets for post-mortems | Structured incident timeline & analytics |

### Incident Lifecycle

```
Detection → Alerting → On-Call Page → Declare Incident
    ↓
Triage → Investigation → Mitigation → Resolution
    ↓
Post-Incident Review → Action Items → Case Management
```

### Core Components

```
Incident Response
├── On-Call
│   ├── Teams          ← Who is on duty
│   ├── Pages          ← Alert notifications sent to on-call
│   └── Send a Page    ← Manually trigger a page
│
├── Incident Management
│   ├── Incident List  ← All active & past incidents
│   ├── Declare Incident ← Open a new incident
│   └── Settings       ← Severity levels, templates, integrations
│
├── Status Pages (NEW) ← External-facing service status
│
└── Case Management    ← Track follow-up actions and bugs
```

---

## 2. Incident Response Navigation Overview

### Accessing Incident Response

```
Datadog Sidebar → Incident Response
```

### Menu Structure

```
On-Call
  ├── Teams       → Manage schedules, rotations, escalation policies
  ├── Settings    → Global on-call configuration
  ├── Pages       → View history of all pages sent
  └── Send a Page → Manually page a team or individual

Incident Management
  ├── Incident List   → Active / resolved incidents dashboard
  ├── Declare Incident → Create a new incident manually
  └── Settings        → Severity config, notification templates

Status Pages  [NEW]
  └── Create public or private status pages per service/product

Case Management
  └── Track bugs, investigations, and follow-up items from incidents
```

---

## 3. On-Call — Teams

### What is an On-Call Team?

An On-Call Team in Datadog is a group of people with a defined **schedule**, **rotation**, and **escalation policy** that determines who gets paged when an alert fires.

### Navigate to: Incident Response → On-Call → Teams

### Team Components

```
Team
├── Members          ← Who belongs to the team
├── Schedule         ← Who is on duty, when
│   ├── Primary rotation
│   └── Override shifts
├── Escalation Policy
│   ├── Level 1: Page primary on-call (0 min)
│   ├── Level 2: Page secondary (5 min, if no ack)
│   └── Level 3: Page team lead (15 min, if no ack)
└── Notification Rules ← How members are notified
```

### Creating a New On-Call Team

**Step 1 — Create the Team**
```
1. Navigate: Incident Response → On-Call → Teams
2. Click "+ New Team"
3. Enter team name (e.g., "Platform SRE")
4. Add team members (search by Datadog username/email)
5. Set team description and tags
```

**Step 2 — Define the Schedule**
```
Schedule types:
  Weekly rotation   ← Each person covers a full week
  Daily rotation    ← Rotate every 24 hours
  Custom rotation   ← Define any interval

Example: Weekly Rotation
  Week 1: Alice
  Week 2: Bob
  Week 3: Charlie
  Week 4: Alice (repeats)
```

**Step 3 — Configure Escalation Policy**
```
Level 1 (immediate):
  → Page on-call member
  → Wait: 5 minutes for acknowledgment

Level 2 (if not acknowledged):
  → Page backup on-call member
  → Wait: 10 minutes

Level 3 (if still not acknowledged):
  → Page team lead + #ops-critical Slack channel
  → Wait: 15 minutes

Final:
  → Escalate to VP Engineering
```

### Schedule Overrides

Use overrides when someone is unavailable:
```
1. Open team schedule
2. Click "Add Override"
3. Select: Who is covering
4. Set: Start time → End time
5. Reason: Optional (vacation, sick leave, etc.)
```

### Notification Rules per Member

Each on-call member can configure how they are notified:

```
Immediately:    Push notification (Datadog mobile app)
After 2 min:    Phone call
After 5 min:    SMS
After 10 min:   Email
```

### On-Call Settings

Navigate to: **Incident Response → On-Call → Settings**

```
Settings:
  Timezone:        UTC+05:30 (IST) — visible in your dashboard
  Business hours:  Define "off-hours" for different escalation paths
  Low urgency:     Separate policy for P4/P5 alerts (no phone calls)
  High urgency:    Full escalation for P1/P2 alerts
```

---

## 4. On-Call — Pages & Escalations

### What is a Page?

A **Page** is a notification sent to the on-call person when a monitor triggers or when manually sent.

### Navigate to: Incident Response → On-Call → Pages

### Page History View

The Pages list shows:

| Column | Description |
|--------|-------------|
| **Status** | Triggered / Acknowledged / Resolved |
| **Title** | Alert name that triggered the page |
| **Team** | Which on-call team was paged |
| **Responder** | Who the page was routed to |
| **Triggered** | Timestamp of the page |
| **Ack time** | How long until someone acknowledged |
| **Duration** | Total time from trigger to resolve |

### Page Lifecycle

```
Monitor fires
     ↓
Page created (status: Triggered)
     ↓
On-call member notified (push, SMS, call, email)
     ↓
Member acknowledges (status: Acknowledged)
     ↓
Member investigates & resolves
     ↓
Monitor recovers (status: Resolved)
```

### Sending a Manual Page

Navigate to: **Incident Response → On-Call → Send a Page**

Use this when:
- You detect an issue before monitors catch it
- You need to escalate immediately
- You're testing your on-call routing

```
Steps:
1. Click "Send a Page"
2. Select team: Platform SRE
3. Priority: P1 / P2 / P3
4. Title: [Manual] Database replication lag detected
5. Message: Replication is 45 seconds behind on db-prod-02
             Immediate investigation required
6. Click "Send"
```

### Acknowledge a Page

From the mobile app or email:
- Tap **"Acknowledge"** to claim ownership
- Prevents further escalation
- Sets your status to active responder

### Linking Pages to Incidents

When you acknowledge a page, you can:
```
[ ] Create a new incident from this page
[ ] Link to existing incident #INC-247
[ ] Resolve without incident (false alarm)
```

---

## 5. Incident Management

### Navigate to: Incident Response → Incident Management → Incident List

### Incident List Dashboard

The Incident List shows all incidents with:

```
┌────────┬──────────────────────────────┬──────────┬────────────┬──────────────┐
│ ID     │ Title                        │ Severity │ Status     │ Duration     │
├────────┼──────────────────────────────┼──────────┼────────────┼──────────────┤
│ INC-01 │ Checkout service down        │ SEV-1    │ Active     │ 00:23:14     │
│ INC-02 │ Elevated 5xx on payments API │ SEV-2    │ Stable     │ 01:05:30     │
│ INC-03 │ Slow queries on prod DB      │ SEV-3    │ Resolved   │ 02:14:00     │
│ INC-04 │ Jenkins build failures       │ SEV-4    │ Resolved   │ 00:45:00     │
└────────┴──────────────────────────────┴──────────┴────────────┴──────────────┘
```

### Filtering the Incident List

```
By Status:   Active / Stable / Resolved / All
By Severity: SEV-1 / SEV-2 / SEV-3 / SEV-4 / SEV-5
By Team:     Platform SRE / Backend / Database / All
By Service:  checkout / payments / auth / All
Date range:  Last 24h / Last 7d / Last 30d / Custom
```

### Incident Detail View

Clicking an incident opens the full incident timeline:

```
Incident #INC-247: Checkout service elevated error rate

[ Overview ]  [ Timeline ]  [ Responders ]  [ Notifications ]  [ Analytics ]

Overview:
  Created:   May 27, 2024 14:32 UTC
  Severity:  SEV-2
  Status:    Active
  Commander: @alice
  Responders: @alice, @bob, @charlie

Timeline:
  14:32 — Incident declared by @alice
  14:33 — @bob acknowledged and joined
  14:35 — Monitor [Checkout] High Error Rate triggered
  14:40 — Identified bad deployment: v2.5.1
  14:45 — Rollback initiated to v2.5.0
  14:52 — Error rate normalizing
  15:01 — Incident resolved
```

---

## 6. Declaring & Managing an Incident

### Navigate to: Incident Response → Incident Management → Declare Incident

### Step-by-Step: Declaring an Incident

**Step 1 — Set Severity & Title**
```
Title:    [Checkout] Elevated error rate - 5xx spike
Severity: SEV-2 — Major Impact (partial service outage)
```

**Step 2 — Describe the Impact**
```
Customer impact:
  Customers unable to complete checkout
  Estimated: 15% of transactions failing
  Affected regions: us-east-1

Detection method:
  ○ Monitor alert
  ● Manual detection
  ○ Customer report
```

**Step 3 — Assign Roles**
```
Incident Commander:  @alice (leads the incident)
Communications Lead: @bob  (updates stakeholders)
Technical Lead:      @charlie (investigates root cause)
```

**Step 4 — Select Affected Services**
```
Services: checkout-api, payment-processor
Environment: production
Teams: payments, platform-sre
```

**Step 5 — Notify Teams**
```
Auto-notify:
  ✅ #incident-response (Slack)
  ✅ PagerDuty — payments team
  ✅ Email — engineering leadership
  ✅ Status page update (if configured)
```

**Step 6 — Declare**
```
Click "Declare Incident" → Incident is now active
```

### Incident Management During Active Incident

#### Adding Timeline Entries

```
14:40 — @alice:
  Root cause identified: bad deployment v2.5.1
  Action: Initiating rollback

14:45 — @charlie:
  Rollback to v2.5.0 in progress
  kubectl rollout undo deployment/checkout-api

14:52 — @bob:
  Error rate dropping: 5.2% → 1.8% → 0.3%
  Monitoring for 5 minutes before resolution
```

#### Updating Incident Status

```
Status transitions:
  Active  → Stable    (issue identified, mitigation in progress)
  Stable  → Resolved  (service fully restored)
  Resolved → Closed   (post-mortem completed)
```

#### Adding Related Signals

From the incident, link:
- Triggered monitors
- Relevant log queries
- Dashboard snapshots
- Runbook URLs

---

## 7. Incident Severity & Classification

### Navigate to: Incident Management → Settings → Severity Levels

### Default Severity Matrix

| Severity | Name | Description | Response SLA | Examples |
|----------|------|-------------|-------------|---------|
| **SEV-1** | Critical | Complete service outage | < 5 minutes | Site down, data loss, security breach |
| **SEV-2** | Major | Significant feature broken | < 15 minutes | Checkout failing, auth errors, payment issues |
| **SEV-3** | Minor | Degraded performance | < 1 hour | Slow queries, high latency, non-critical errors |
| **SEV-4** | Low | Minor issue | < 4 hours | Build failures, low-priority alerts |
| **SEV-5** | Info | Informational | Best effort | Planned maintenance, capacity warnings |

### Customizing Severity Levels

In Settings, you can:
```
1. Rename severity levels to match your org's language
   (e.g., P0/P1/P2 instead of SEV-1/SEV-2/SEV-3)

2. Define criteria for each level:
   - User impact threshold (% of users affected)
   - Revenue impact threshold ($X/min)
   - SLA breach conditions

3. Set automatic escalation rules:
   - SEV-1: Auto-page CEO, CTO
   - SEV-2: Auto-page VP Engineering
   - SEV-3: Page on-call team only
```

### Severity Decision Tree

```
Is the entire service down?
  YES → SEV-1
  NO  ↓

Are customers unable to complete core workflows?
  YES → SEV-2
  NO  ↓

Is performance significantly degraded (>30% slower)?
  YES → SEV-3
  NO  ↓

Is it a minor issue with minimal user impact?
  YES → SEV-4
  NO  → SEV-5 (informational)
```

---

## 8. Status Pages (NEW)

### Navigate to: Incident Response → Status Pages

### What are Status Pages?

Status Pages provide **external-facing communication** about service health — keeping customers and stakeholders informed without flooding them with technical details.

### Types of Status Pages

```
Public Status Page:
  - Accessible by anyone (no login required)
  - URL: status.yourcompany.com
  - Use for: Customer-facing services

Private Status Page:
  - Accessible by authenticated users only
  - Use for: Internal tools, partner services
```

### Creating a Status Page

**Step 1 — Basic Setup**
```
Name:       MyApp Status
URL slug:   status.myapp.com
Logo:       Upload company logo
Theme:      Light / Dark / Custom brand colors
```

**Step 2 — Add Service Components**
```
Components to monitor:
  ✅ API Gateway          (linked to: monitor #123)
  ✅ Web Application      (linked to: synthetic test)
  ✅ Authentication       (linked to: monitor #124)
  ✅ Database             (linked to: monitor #125)
  ✅ Payment Processing   (linked to: monitor #126)
  ✅ CDN / Static Assets  (linked to: synthetic test)
```

**Step 3 — Define Automation Rules**
```
When monitor enters ALERT state:
  → Set component status to: "Degraded Performance"
  → Auto-post incident update to status page

When monitor enters ALERT for > 10 minutes:
  → Escalate to: "Partial Outage"
  → Notify subscribed users via email

When monitor recovers:
  → Set component to: "Operational"
  → Post: "Issue resolved" update
```

### Status Component States

| State | Color | Meaning |
|-------|-------|---------|
| **Operational** | 🟢 Green | All systems normal |
| **Degraded Performance** | 🟡 Yellow | Slower than expected |
| **Partial Outage** | 🟠 Orange | Some users affected |
| **Major Outage** | 🔴 Red | All users affected |
| **Under Maintenance** | 🔵 Blue | Planned downtime |

### Posting Incident Updates

During an active incident:
```
1. Open your Status Page
2. Click "Create Incident"
3. Title: Investigating elevated error rates
4. Affected components: API Gateway, Payment Processing
5. Status: Investigating
6. Message: We are aware of issues affecting checkout
            and are actively investigating.

Updates during incident:
  Update 1: "Identified root cause — deploying fix"
  Update 2: "Fix deployed — monitoring recovery"
  Update 3: "All systems operational — incident resolved"
```

### Subscriber Notifications

Users can subscribe to your status page:
```
Notification options:
  Email         → Updates sent to their inbox
  RSS feed      → For technical users
  Webhook       → For automated integrations
  Slack (beta)  → Direct channel notifications
```

---

## 9. Case Management

### Navigate to: Incident Response → Case Management

### What is Case Management?

**Cases** are structured work items for tracking follow-up actions, investigations, and bugs that arise from incidents or are detected proactively.

### Case vs Incident

| | Case | Incident |
|-|------|---------|
| **Purpose** | Track a work item or investigation | Manage an active outage |
| **Urgency** | Low-medium (can be async) | High (real-time response) |
| **Lifecycle** | Open → In Progress → Closed | Active → Stable → Resolved |
| **Audience** | Engineering team | All stakeholders |
| **SLA** | Planned sprint/sprint work | Minutes to hours |

### Case Types

```
Bug          ← Reproducible defect requiring a fix
Investigation ← Unclear issue requiring analysis
Request      ← Feature request or improvement
Task         ← General follow-up action item
```

### Creating a Case

```
1. Navigate: Incident Response → Case Management → New Case
2. Fill in:
   Type:        Investigation
   Title:       Root cause analysis - Jenkins high error rate
   Description: Jenkins monitors showing NO DATA and DRAFT status.
                Need to verify agent connectivity and metric pipeline.
   Priority:    P3
   Assignee:    @charlie
   Team:        Platform SRE
   Tags:        service:jenkins, env:prod
   Related:     Incident #INC-244, Monitor #456
3. Click "Create Case"
```

### Case from Incident

During an incident, create cases for action items:
```
In incident #INC-247 timeline:
  Post: "Action items for post-incident"
  → Case 1: "Fix memory leak in checkout-api v2.5.1" (Bug, @charlie)
  → Case 2: "Add canary deployment to prevent bad rollouts" (Task, @alice)
  → Case 3: "Improve error rate alerting threshold" (Task, @bob)
```

### Case Workflow

```
New → Assigned → In Progress → In Review → Closed
         ↓
      Blocked (dependency on another team/case)
```

### Linking Cases to Signals

Attach relevant observability context:
```
Linked signals:
  📊 Dashboard: Checkout Service Health
  📋 Monitor:   [Checkout] High Error Rate
  📝 Logs:      Error logs during incident window
  🔗 Runbook:   https://wiki.internal/checkout-runbook
  📌 PR/Commit: github.com/org/checkout/pull/892
```

---

## 10. Incident Workflows & Automation

### Setting Up Automated Workflows

Navigate to: **Incident Management → Settings → Workflows**

### Workflow Triggers

```
Trigger: Incident declared with severity SEV-1 or SEV-2
Actions:
  1. Post to #incident-response Slack channel
  2. Page on-call team (if not already paged)
  3. Create Jira ticket for tracking
  4. Update status page to "Investigating"
  5. Send email to engineering leadership
  6. Start 15-minute check-in reminder
```

### Sample Workflow: SEV-1 Auto-Response

```yaml
name: SEV-1 Auto Response
trigger:
  event: incident_declared
  condition: severity == "SEV-1"

actions:
  - type: slack_message
    channel: "#incidents-critical"
    message: |
      🔴 SEV-1 INCIDENT DECLARED
      Title: {{incident.title}}
      Commander: {{incident.commander}}
      War Room: {{zoom_link}}

  - type: page_team
    team: "Platform SRE"
    priority: P1

  - type: create_jira_ticket
    project: INCIDENT
    priority: Critical

  - type: update_status_page
    component: "API Gateway"
    status: "Major Outage"
    message: "We are investigating a critical issue"

  - type: schedule_reminder
    interval: 15_minutes
    message: "Time for a status update — post in the incident channel"
```

### Runbook Integration

Link runbooks to monitors and incidents:
```
Monitor settings → Runbook URL:
  https://wiki.internal/runbooks/checkout-errors
  https://wiki.internal/runbooks/database-issues
  https://wiki.internal/runbooks/jenkins-ci

Runbook appears in:
  - Monitor alert notification
  - Incident detail page
  - On-call page notification
```

---

## 11. Post-Incident Review

### Post-Mortem Process

After resolving an SEV-1 or SEV-2 incident, conduct a **blameless post-mortem**:

```
Timeline (from Datadog incident):
  Use the auto-generated incident timeline as the base

Document structure:
  1. Incident Summary
  2. Timeline of Events
  3. Root Cause Analysis (5 Whys)
  4. Contributing Factors
  5. Impact Assessment
  6. What Went Well
  7. What Could Be Improved
  8. Action Items (→ Cases in Case Management)
```

### Post-Mortem Template

```markdown
## Incident Post-Mortem: [Title]
**Incident ID:** INC-XXX
**Date:** YYYY-MM-DD
**Severity:** SEV-X
**Duration:** X hours Y minutes
**Commander:** @name

### Summary
Brief description of what happened and its impact.

### Timeline
| Time (UTC) | Event |
|------------|-------|
| HH:MM | Incident began |
| HH:MM | Alert fired |
| HH:MM | On-call paged |
| HH:MM | Root cause identified |
| HH:MM | Mitigation applied |
| HH:MM | Service restored |

### Root Cause
**Proximate cause:** What directly caused the incident
**Root cause:**     The underlying reason it happened

### 5 Whys Analysis
1. Why did users see errors?        → Checkout API returning 500s
2. Why was the API returning 500s?  → Database connection pool exhausted
3. Why was the pool exhausted?      → Memory leak in v2.5.1
4. Why was the leak not caught?     → No memory testing in CI pipeline
5. Why was there no memory test?    → Not in our test coverage standards

### Impact
- Duration: 29 minutes
- Users affected: ~8,400 (15% of active sessions)
- Revenue impact: ~$12,000 in lost transactions
- SLO impact: 99.94% monthly availability (budget: 43.2 min)

### Action Items
| Item | Owner | Due Date | Case |
|------|-------|----------|------|
| Fix memory leak in v2.5.2 | @charlie | 2024-03-22 | CASE-891 |
| Add memory profiling to CI | @alice | 2024-03-29 | CASE-892 |
| Add canary deployment gate | @bob | 2024-04-05 | CASE-893 |
| Lower error rate alert threshold | @alice | 2024-03-20 | CASE-894 |
```

### Incident Analytics

Navigate to: **Incident Management → Analytics**

Review:
```
MTTD  (Mean Time to Detect)   — How fast monitors caught it
MTTA  (Mean Time to Acknowledge) — How fast on-call responded
MTTM  (Mean Time to Mitigate)  — How fast issue was contained
MTTR  (Mean Time to Resolve)   — How fast full recovery occurred

Trends over time:
  Incidents per week/month
  Breakdown by severity
  Breakdown by service/team
  Recurrence rate (same root cause appearing again)
```

---

## 12. Best Practices

### On-Call Health

```
✅ Rotate on-call shifts fairly — no permanent on-call engineers
✅ Define clear escalation policies — nobody should be unreachable
✅ Set up mobile app notifications for immediate response
✅ Always define a backup — primary + secondary on-call
✅ Document overrides promptly when schedule changes
✅ Review on-call load monthly — alert fatigue is real
✅ P1/P2 = phone call + SMS; P4/P5 = email only
```

### Incident Hygiene

```
✅ Declare early — better to call off than delay response
✅ Assign an Incident Commander for any SEV-1/SEV-2
✅ Post timeline updates every 15 minutes during active incidents
✅ Link related monitors and dashboards immediately
✅ Use status pages to reduce inbound support volume
✅ Always write a post-mortem for SEV-1 and SEV-2
✅ Create Cases for every action item — don't rely on memory
```

### Alert Quality

```
✅ Every monitor should have a runbook URL
✅ High-urgency monitors → phone call escalation
✅ Low-urgency monitors → Slack / email only
✅ Tune thresholds to reduce false pages
✅ Review page history monthly — fix noisy monitors
✅ Use composite monitors to reduce duplicate pages
```

---

## 13. Hands-On Labs

### Lab 1 — Create an On-Call Team (20 min)

**Objective:** Set up a complete on-call rotation with escalation policy.

```
Steps:
1. Navigate: Incident Response → On-Call → Teams
2. Click "+ New Team"
3. Name: Workshop-SRE-Team
4. Add yourself as a member
5. Create a weekly rotation schedule
6. Configure escalation policy:
   Level 1: Page primary on-call (0 min)
   Level 2: Page Slack channel (5 min, no ack)
7. Set notification rules:
   Immediate: Push notification
   After 2 min: Email
8. Save team
```

**Verify:** Team appears in On-Call Teams list with schedule visible.

---

### Lab 2 — Send a Manual Page (10 min)

**Objective:** Practice triggering and acknowledging a page.

```
Steps:
1. Navigate: Incident Response → On-Call → Send a Page
2. Select team: Workshop-SRE-Team
3. Priority: P3
4. Title: [Test] Lab 2 - Manual page test
5. Message: This is a workshop test page. Please acknowledge.
6. Click "Send"
7. Find the page in: On-Call → Pages
8. Click "Acknowledge" on your test page
9. Note: Ack time recorded in page history
```

**Verify:** Page shows "Acknowledged" status in Pages history.

---

### Lab 3 — Declare a Practice Incident (25 min)

**Objective:** Experience the full incident declaration and management flow.

```
Scenario: Your checkout service is showing elevated error rates.

Steps:
1. Navigate: Incident Response → Declare Incident
2. Fill in:
   Title:    [Lab] Checkout service elevated errors
   Severity: SEV-3
   Services: checkout (or any service in your environment)
   Impact:   "Customers seeing intermittent checkout failures"
3. Assign yourself as Incident Commander
4. Enable Slack notification (if integrated)
5. Click "Declare Incident"

During the incident:
6. Add 3 timeline entries:
   Entry 1: "Investigating — reviewing error logs"
   Entry 2: "Identified cause: high DB query time"
   Entry 3: "Mitigation applied: scaled DB read replicas"
7. Change status from Active → Stable → Resolved
8. View the auto-generated incident timeline
```

**Verify:** Incident appears in Incident List as "Resolved" with full timeline.

---

### Lab 4 — Set Up a Status Page (20 min)

**Objective:** Create a status page and simulate an incident update.

```
Steps:
1. Navigate: Incident Response → Status Pages → New Status Page
2. Configure:
   Name:   Workshop Status Page
   Type:   Private (for lab purposes)
3. Add components:
   - API Service
   - Web Application
   - Database
4. Set all components to: "Operational"
5. Simulate an incident:
   Click "Create Incident" on the status page
   Title: Investigating API performance issues
   Affected: API Service
   Status: Investigating
   Message: We are investigating reports of slow API responses.
6. Post an update:
   Message: The issue has been identified and a fix is being deployed.
   Status: Identified
7. Resolve:
   Message: The issue has been resolved. All systems operational.
   Status: Resolved
```

**Verify:** Status page shows incident history with all updates visible.

---

### Lab 5 — Create Cases from an Incident (15 min)

**Objective:** Practice converting incident action items into tracked cases.

```
Prerequisites: Complete Lab 3 (your resolved incident)

Steps:
1. Open your resolved incident from Lab 3
2. Navigate to the "Action Items" or "Cases" section
3. Create 2 cases:

   Case 1:
     Type: Investigation
     Title: Analyze DB query performance patterns
     Priority: P3
     Assignee: Yourself
     Tag: service:checkout, env:prod

   Case 2:
     Type: Task
     Title: Add DB connection pool monitoring
     Priority: P4
     Assignee: Yourself
     Link to: Your Lab 3 incident

4. Navigate: Incident Response → Case Management
5. Find both cases in the list
6. Move Case 1 to "In Progress" status
```

**Verify:** Both cases appear in Case Management linked to your incident.

---

## Appendix

### Key Metrics to Track

```
MTTD  = Time from incident start → monitor alert
MTTA  = Time from alert → on-call acknowledgment
MTTM  = Time from ack → mitigation applied
MTTR  = Time from ack → full resolution
Escalation rate = Pages that required Level 2+ escalation
False positive rate = Pages that were false alarms
```

### Incident Response Roles

| Role | Responsibilities |
|------|-----------------|
| **Incident Commander** | Owns the incident, coordinates response, makes decisions |
| **Technical Lead** | Investigates root cause, applies fixes |
| **Communications Lead** | Updates stakeholders, posts status page updates |
| **Scribe** | Documents timeline, decisions, and action items |
| **Subject Matter Expert** | Called in for specific system knowledge |

### Useful Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Global search — find any incident, monitor, or team |
| `G I` | Go to Incident List |

### Glossary

| Term | Definition |
|------|-----------|
| **Blameless Post-Mortem** | A review focused on systems & processes, not blaming individuals |
| **Escalation Policy** | Rules for who gets paged if the primary responder doesn't acknowledge |
| **MTTD** | Mean Time to Detect — how long before an issue is noticed |
| **MTTR** | Mean Time to Resolve — total response time |
| **On-Call Rotation** | Schedule defining who is responsible at any given time |
| **Override** | Temporary schedule change (e.g., swapping shifts) |
| **Page** | A notification sent to the on-call person |
| **Runbook** | Step-by-step guide for responding to a specific alert |
| **SLO** | Service Level Objective — reliability target for a service |
| **War Room** | A dedicated (virtual) space for incident responders to collaborate |

---

### Workshop Resources

- 📘 On-Call Docs: https://docs.datadoghq.com/service_management/on-call/
- 🚨 Incident Management Docs: https://docs.datadoghq.com/service_management/incident_management/
- 📄 Status Pages Docs: https://docs.datadoghq.com/service_management/status_page/
- 📋 Case Management Docs: https://docs.datadoghq.com/service_management/case_management/
- 🎓 Datadog Learning Center: https://learn.datadoghq.com

---

*Workshop Version 1.0 | Datadog Incident Response Training Guide*
