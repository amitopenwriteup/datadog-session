# Datadog Administration Workshop
### Free Trial + AWS EC2 | Access Management & Governance

---

## Prerequisites

Before starting, ensure you have:

- A Datadog free trial account → [datadoghq.com](https://www.datadoghq.com)
- An AWS account with at least one EC2 instance (Amazon Linux 2 or Ubuntu 22.04 recommended)
- SSH access to your EC2 instance
- `curl` and `bash` available on the EC2 instance
- Your Datadog **API Key** and **Site** (e.g., `datadoghq.com`)

---

## Workshop 1 — Creating and Managing Datadog Organizations

### What Is a Datadog Organization?

A Datadog **organization (org)** is a top-level isolated tenant. All users, data, dashboards, monitors, and keys belong to an org. On free trial, you get one primary org but can create child orgs for multi-tenant practice.

### Step-by-Step: Exploring Your Org

1. Log in to [app.datadoghq.com](https://app.datadoghq.com)
2. Navigate to **Organization Settings** → **Profile**
3. Note your:
   - **Org Name**
   - **Org ID** (used in API calls)
   - **Datadog Site** (e.g., `US1`, `EU`)

### Step-by-Step: Create a Child Organization (Trial Limitation Note)

> ⚠️ Child org creation requires Datadog **Enterprise** plan. On Free Trial, simulate this using the API to understand the structure.

```bash
# On your EC2 instance — create a child org via API (Enterprise accounts)
curl -X POST "https://api.datadoghq.com/api/v1/org" \
  -H "DD-API-KEY: <YOUR_API_KEY>" \
  -H "DD-APPLICATION-KEY: <YOUR_APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Workshop-ChildOrg-01",
    "subscription": {"type": "pro"},
    "billing": {"type": "parent_billing"}
  }'
```

### Step-by-Step: List Organizations

```bash
# List all orgs associated with your account
curl -X GET "https://api.datadoghq.com/api/v1/org" \
  -H "DD-API-KEY: <YOUR_API_KEY>" \
  -H "DD-APPLICATION-KEY: <YOUR_APP_KEY>"
```

### Step-by-Step: Switch Between Orgs (UI)

1. Click your **avatar** (top-right corner)
2. Select **Switch Organization**
3. Choose the org from the dropdown

### Best Practices

- Use separate orgs for **Production**, **Staging**, and **Development** environments
- Name orgs clearly: `company-prod`, `company-dev`
- Designate at least **2 admins** per org for redundancy
- Never use a personal account as the sole org owner

---

## Workshop 2 — API Keys vs Application Keys

### Concept Overview

| Feature | API Key | Application Key |
|---|---|---|
| **Purpose** | Authenticate data ingestion (agents, logs, metrics) | Authenticate API management actions |
| **Scope** | Org-wide | Scoped to the creating user's permissions |
| **Used by** | Datadog Agent, AWS integration, CI pipelines | Scripts, automation, dashboards |
| **Revocable** | Yes | Yes |
| **Secret** | Yes — treat like a password | Yes — treat like a password |

### Step-by-Step: Create an API Key (UI)

1. Go to **Organization Settings** → **API Keys**
2. Click **+ New Key**
3. Name it descriptively: `ec2-workshop-agent-key`
4. Copy and store it immediately — it is shown only once

### Step-by-Step: Create an API Key (API)

```bash
curl -X POST "https://api.datadoghq.com/api/v2/api_keys" \
  -H "DD-API-KEY: <EXISTING_API_KEY>" \
  -H "DD-APPLICATION-KEY: <YOUR_APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"data": {"type": "api_keys", "attributes": {"name": "ec2-agent-key-01"}}}'
```

### Step-by-Step: Create an Application Key (UI)

1. Go to **Organization Settings** → **Application Keys**
2. Click **+ New Key**
3. Name it: `workshop-automation-key`
4. **Scope it** — select only required scopes (e.g., `dashboards_read`, `monitors_write`)
5. Copy and store securely

### Step-by-Step: Install Datadog Agent on EC2 Using Your API Key

```bash
# SSH into your EC2 instance
ssh -i your-key.pem ec2-user@<EC2_PUBLIC_IP>

# One-line Datadog Agent install (replace DD_API_KEY and DD_SITE)
DD_API_KEY=<YOUR_API_KEY> \
DD_SITE="datadoghq.com" \
bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script_agent7.sh)"

# Verify agent is running
sudo systemctl status datadog-agent

# Check agent status
sudo datadog-agent status
```

### Step-by-Step: Verify Key in Agent Config

```bash
sudo cat /etc/datadog-agent/datadog.yaml | grep api_key
```

### Step-by-Step: Revoke an API Key

```bash
# Get the key ID first
curl -X GET "https://api.datadoghq.com/api/v2/api_keys" \
  -H "DD-API-KEY: <KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"

# Delete by ID
curl -X DELETE "https://api.datadoghq.com/api/v2/api_keys/<KEY_ID>" \
  -H "DD-API-KEY: <KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"
```

### Best Practices

- **One key per service** — never share keys across systems
- **Label keys** with: service name, environment, creation date (e.g., `prod-ec2-agent-2025-06`)
- **Rotate keys every 90 days**
- **Never commit keys** to Git — use AWS Secrets Manager or environment variables
- **Scope Application Keys** to minimum required permissions

---

## Workshop 3 — Role-Based Access Control (RBAC)

### Built-in Roles

Datadog provides three built-in roles:

| Role | Description |
|---|---|
| **Datadog Admin Role** | Full access — manage users, keys, billing, all settings |
| **Datadog Standard Role** | Read/write access to monitoring resources, no admin settings |
| **Datadog Read Only Role** | View-only access to dashboards, monitors, logs |

### Step-by-Step: Assign a Built-in Role to a User (UI)

1. Go to **Organization Settings** → **Users**
2. Find the user or click **Invite Users**
3. Under the user's **Roles** column, click **Edit**
4. Select a built-in role from the dropdown
5. Click **Save**

### Step-by-Step: Create a Custom Role (UI)

1. Go to **Organization Settings** → **Roles**
2. Click **+ New Role**
3. Name it: `EC2-Monitoring-Viewer`
4. Add permissions:
   - `metrics_read`
   - `dashboards_read`
   - `infrastructure_list`
5. Click **Save**

### Step-by-Step: Create a Custom Role (API)

```bash
curl -X POST "https://api.datadoghq.com/api/v2/roles" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "type": "roles",
      "attributes": {"name": "EC2-Monitoring-Viewer"},
      "relationships": {
        "permissions": {
          "data": [
            {"type": "permissions", "id": "<PERMISSION_ID_metrics_read>"},
            {"type": "permissions", "id": "<PERMISSION_ID_dashboards_read>"}
          ]
        }
      }
    }
  }'
```

### Step-by-Step: List All Permissions

```bash
curl -X GET "https://api.datadoghq.com/api/v2/permissions" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"
```

### Step-by-Step: Assign Custom Role to User

```bash
# Assign role to user
curl -X POST "https://api.datadoghq.com/api/v2/roles/<ROLE_ID>/users" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {"type": "users", "id": "<USER_ID>"}
  }'
```

### Best Practices

- Apply **Principle of Least Privilege** — start with Read Only, escalate as needed
- **Avoid using Admin Role** for daily operational tasks
- **Audit role assignments** quarterly
- Use **custom roles** for team-specific access (DevOps, Security, Finance)
- Document role definitions in a central runbook

---

## Workshop 4 — Teams and User Management

### What Are Datadog Teams?

Teams are logical groupings of users that can own dashboards, monitors, services, and incidents. Teams improve visibility and on-call routing.

### Step-by-Step: Create a Team (UI)

1. Go to **Organization Settings** → **Teams**
2. Click **+ New Team**
3. Fill in:
   - **Name**: `AWS-EC2-Ops`
   - **Handle**: `aws-ec2-ops` (auto-generated)
   - **Description**: `Team managing EC2 infrastructure monitoring`
4. Click **Create**

### Step-by-Step: Add Members to a Team

1. Open the team `AWS-EC2-Ops`
2. Click **Add Team Members**
3. Search for users and assign roles:
   - `Team Manager` — can manage team membership
   - `Team Member` — standard access

### Step-by-Step: Invite a New User (UI)

1. Go to **Organization Settings** → **Users**
2. Click **Invite Users**
3. Enter the email address
4. Select role(s): e.g., `Datadog Standard Role` + custom `EC2-Monitoring-Viewer`
5. Click **Send Invite**

### Step-by-Step: Manage Users via API

```bash
# List all users
curl -X GET "https://api.datadoghq.com/api/v2/users" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"

# Disable a user
curl -X DELETE "https://api.datadoghq.com/api/v2/users/<USER_ID>" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"
```

### Step-by-Step: Assign a Team to a Dashboard

1. Open any dashboard
2. Click **⚙️ Settings** → **Edit Dashboard**
3. Under **Teams**, type `AWS-EC2-Ops` and select it
4. Save the dashboard

### Best Practices

- Create teams **per service or domain** (e.g., `Payments`, `Auth`, `Infrastructure`)
- Assign team ownership to monitors for **automatic alert routing**
- Review team membership during **employee offboarding**
- Use **team handles** in monitor messages: `@team-aws-ec2-ops`

---

## Workshop 5 — SSO and SAML Integration

### Concept Overview

SSO (Single Sign-On) via SAML allows your identity provider (IdP — e.g., Okta, Azure AD, Google Workspace) to authenticate users into Datadog without separate Datadog passwords.

> ⚠️ SAML SSO requires **Datadog Pro or Enterprise** plan. On Free Trial, you can view the configuration screen but cannot fully activate. Use this workshop to understand the setup flow.

### Supported Identity Providers

- Okta
- Azure Active Directory
- Google Workspace
- OneLogin
- Generic SAML 2.0

### Step-by-Step: Access SAML Settings (UI)

1. Go to **Organization Settings** → **SAML**
2. Note the **Datadog Service Provider metadata URL** — provide this to your IdP
3. Fields you will need from your IdP:
   - **Identity Provider Single Sign-On URL**
   - **Identity Provider Issuer/Entity ID**
   - **x.509 Certificate**

### Step-by-Step: Configure SAML with Okta (Overview)

**In Okta:**
```
1. Create a new SAML 2.0 Application in Okta
2. Single Sign-On URL: https://app.datadoghq.com/account/saml/assertion
3. Audience URI (SP Entity ID): https://app.datadoghq.com/account/saml/metadata.xml
4. Name ID Format: EmailAddress
5. Attribute mapping:
   - Email → user.email
   - FirstName → user.firstName
   - LastName → user.lastName
6. Download Okta metadata XML
```

**In Datadog:**
```
1. Go to Organization Settings → SAML
2. Upload the Okta metadata XML
3. Set "Just-in-Time Provisioning" to ON (auto-creates users on first login)
4. Map IdP groups to Datadog roles via SAML attributes
5. Enable SAML
```

### Step-by-Step: Test SSO Login

```bash
# Open in browser (replace YOUR_ORG)
https://app.datadoghq.com/account/login/id/<YOUR_ORG_ID>
```

### SAML Attribute Mapping for Role Assignment

```xml
<!-- Example SAML assertion attribute for role mapping -->
<saml:Attribute Name="roles">
  <saml:AttributeValue>Datadog Standard Role</saml:AttributeValue>
</saml:Attribute>
```

Configure in Datadog under **SAML** → **Role Mappings** to map IdP groups → Datadog roles.

### Best Practices

- Enable **Just-in-Time (JIT) Provisioning** to auto-create users
- Disable **direct password login** after SSO is validated
- Map **IdP groups to Datadog roles** for automatic access control
- Test SSO with a **non-admin account** before enforcing org-wide
- Configure **SSO bypass** (emergency access) for at least one admin

---

## Workshop 6 — Audit Trail for Account-Level Events

### What Is the Audit Trail?

The Datadog **Audit Trail** logs all account-level actions: who changed what, when. This is critical for compliance (SOC2, PCI-DSS, HIPAA) and security investigations.

> ✅ Audit Trail is available on **Free Trial** for exploration.

### Step-by-Step: Enable Audit Trail

1. Go to **Organization Settings** → **Audit Trail**
2. Toggle **Enable Audit Trail** → ON
3. Set retention: 7 days (trial), up to 90 days (paid plans)

### Step-by-Step: Access Audit Events (UI)

1. Navigate to **Security** → **Audit Trail** (or search "Audit Trail" in the nav)
2. Use filters:
   - **User**: filter by specific user email
   - **Event Type**: `api_key.created`, `user.login`, `dashboard.deleted`
   - **Time Range**: last 1 hour, 24 hours, 7 days

### Key Audit Event Types to Monitor

| Event | Why It Matters |
|---|---|
| `api_key.created` | Track new key creation |
| `api_key.deleted` | Key revocation events |
| `user.login` | Authentication monitoring |
| `user.invitation_sent` | New user provisioning |
| `role.created` / `role.updated` | RBAC changes |
| `org_settings.updated` | Configuration drift |
| `dashboard.deleted` | Resource deletion |
| `monitor.muted` | Suppression of alerts |

### Step-by-Step: Query Audit Events via API

```bash
# Get audit events from the last hour
curl -X GET "https://api.datadoghq.com/api/v2/audit/events" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>" \
  -G \
  --data-urlencode "filter[query]=@evt.name:api_key.created" \
  --data-urlencode "filter[from]=2025-01-01T00:00:00Z" \
  --data-urlencode "filter[to]=2025-12-31T23:59:59Z" \
  --data-urlencode "page[limit]=50"
```

### Step-by-Step: Forward Audit Logs to S3 for Long-Term Storage

```bash
# On EC2 — configure Datadog Agent to forward audit logs to S3
# 1. Create an S3 bucket: datadog-audit-logs-<account-id>
# 2. In Datadog: Integrations → AWS → Log Forwarding
# 3. Or use Datadog Log Archives: Logs → Configuration → Archives
#    Set source: Audit Trail, destination: S3 bucket ARN
```

### Step-by-Step: Create a Monitor on Audit Events

```bash
# Alert on suspicious: 5+ failed logins in 5 minutes
curl -X POST "https://api.datadoghq.com/api/v1/monitor" \
  -H "DD-API-KEY: <API_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alert: Multiple Failed Logins",
    "type": "log alert",
    "query": "logs(\"source:audit @evt.name:user.login @action:failed\").index(\"*\").rollup(\"count\").last(\"5m\") > 5",
    "message": "⚠️ Multiple failed login attempts detected. @team-aws-ec2-ops",
    "tags": ["security", "audit"],
    "options": {"notify_no_data": false, "thresholds": {"critical": 5}}
  }'
```

### Best Practices

- Enable Audit Trail **immediately** after creating the org
- Archive audit logs to **S3 or GCS** for 12+ month retention
- Set up **monitors on security-sensitive events** (key creation, role changes, failed logins)
- Review audit logs **weekly** as part of security operations
- Restrict Audit Trail access to **Admin and Security roles** only

---

## Workshop 7 — Key Rotation and Access Governance Best Practices

### Key Rotation Strategy

#### Rotation Schedule

| Key Type | Rotation Frequency | Trigger Rotation Immediately If |
|---|---|---|
| API Keys (production) | Every 90 days | Employee leaves, key leaked |
| API Keys (dev/test) | Every 180 days | Repo made public, key seen in logs |
| Application Keys | Every 90 days | User role change, compromised |
| SAML Certificates | Per IdP policy (1 year) | IdP breach |

### Step-by-Step: Rotate an API Key (Zero-Downtime)

```bash
# Step 1: Create a new API key
NEW_KEY=$(curl -s -X POST "https://api.datadoghq.com/api/v2/api_keys" \
  -H "DD-API-KEY: <CURRENT_KEY>" \
  -H "DD-APPLICATION-KEY: <APP_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"data": {"type": "api_keys", "attributes": {"name": "ec2-agent-key-rotated-$(date +%Y%m%d)"}}}' \
  | jq -r '.data.attributes.key')

echo "New API Key: $NEW_KEY"

# Step 2: Update Datadog Agent on EC2 with new key
ssh -i key.pem ec2-user@<EC2_IP> "
  sudo sed -i 's/api_key:.*/api_key: $NEW_KEY/' /etc/datadog-agent/datadog.yaml
  sudo systemctl restart datadog-agent
  sudo datadog-agent status | grep -i 'api key'
"

# Step 3: Verify agent is submitting metrics
# Wait 2-3 minutes, then check in Datadog Infrastructure List

# Step 4: Revoke the old key
curl -X DELETE "https://api.datadoghq.com/api/v2/api_keys/<OLD_KEY_ID>" \
  -H "DD-API-KEY: $NEW_KEY" \
  -H "DD-APPLICATION-KEY: <APP_KEY>"
```

### Step-by-Step: Store API Keys Securely on EC2 Using AWS Secrets Manager

```bash
# Store key in AWS Secrets Manager
aws secretsmanager create-secret \
  --name "datadog/api-key/prod-ec2" \
  --description "Datadog API Key for prod EC2 agents" \
  --secret-string '{"api_key":"<YOUR_API_KEY>"}' \
  --region ap-south-1

# Retrieve key in a script
API_KEY=$(aws secretsmanager get-secret-value \
  --secret-id "datadog/api-key/prod-ec2" \
  --query 'SecretString' --output text | jq -r '.api_key')

# Use in agent config update
sudo sed -i "s/api_key:.*/api_key: $API_KEY/" /etc/datadog-agent/datadog.yaml
```

### Step-by-Step: Access Governance Checklist

```bash
# Run this quarterly access review script on EC2
#!/bin/bash
echo "=== Datadog Access Governance Review ==="
echo "Date: $(date)"
echo ""

# 1. List all API keys
echo "--- API Keys ---"
curl -s -X GET "https://api.datadoghq.com/api/v2/api_keys" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | jq '.data[] | {name: .attributes.name, last_used: .attributes.last4, created: .attributes.created_at}'

# 2. List all users
echo "--- Users ---"
curl -s -X GET "https://api.datadoghq.com/api/v2/users" \
  -H "DD-API-KEY: $DD_API_KEY" \
  -H "DD-APPLICATION-KEY: $DD_APP_KEY" \
  | jq '.data[] | {email: .attributes.email, status: .attributes.status, roles: .relationships.roles}'

# 3. Identify keys older than 90 days (manual review)
echo ""
echo "ACTION REQUIRED: Review above list and revoke:"
echo "  - API keys unused for 90+ days"
echo "  - Keys owned by departed employees"
echo "  - Application keys with excessive scopes"
```

### Access Governance Framework

```
QUARTERLY REVIEW CHECKLIST
===========================

[ ] Review all API keys — remove unused/orphaned keys
[ ] Review all Application Keys — verify scope is minimal
[ ] Review user list — disable accounts for departed employees
[ ] Review role assignments — verify Principle of Least Privilege
[ ] Review team memberships — remove stale members
[ ] Check Audit Trail for anomalies — failed logins, bulk deletes
[ ] Rotate all production API keys
[ ] Review SAML role mappings — verify IdP group→Datadog role accuracy
[ ] Test SSO login flow end-to-end
[ ] Update key inventory document with new key names and expiry dates
```

### Summary of Best Practices

| Area | Best Practice |
|---|---|
| **API Keys** | One key per service; label with date; rotate every 90 days |
| **App Keys** | Scope to minimum permissions; owned by service accounts, not humans |
| **Roles** | Least privilege; no shared admin accounts; custom roles for teams |
| **Users** | SSO-only login in production; JIT provisioning; immediate offboarding |
| **Teams** | Align to services/domains; assign monitor ownership to teams |
| **Audit Trail** | Always enabled; archived to S3; monitored with alerts |
| **Key Storage** | AWS Secrets Manager or HashiCorp Vault; never in code or `.env` files |
| **Offboarding** | Disable user → revoke all keys → remove from teams → audit |

---

## Quick Reference: Useful API Endpoints

```bash
# Base URL
BASE="https://api.datadoghq.com"

# Authentication headers
HEADERS='-H "DD-API-KEY: $DD_API_KEY" -H "DD-APPLICATION-KEY: $DD_APP_KEY"'

# Endpoints
GET  $BASE/api/v2/api_keys          # List API keys
POST $BASE/api/v2/api_keys          # Create API key
DEL  $BASE/api/v2/api_keys/{id}     # Delete API key
GET  $BASE/api/v2/users             # List users
POST $BASE/api/v2/users             # Invite user
GET  $BASE/api/v2/roles             # List roles
POST $BASE/api/v2/roles             # Create role
GET  $BASE/api/v2/permissions       # List all permissions
GET  $BASE/api/v2/audit/events      # Query audit trail
GET  $BASE/api/v1/org               # Get org info
```

---

## Troubleshooting

| Issue | Resolution |
|---|---|
| Agent not sending data | Check API key: `sudo datadog-agent status` |
| 403 on API call | Verify App Key has required scope |
| SAML login fails | Check certificate expiry and ACS URL |
| User not receiving invite | Check spam folder; verify email domain is not blocklisted |
| Audit Trail empty | Ensure it is enabled in Organization Settings |
| Key rotation breaks agent | Always create new key before deleting old; verify agent restarts cleanly |

---

*Workshop Guide | Datadog Free Trial + AWS EC2 | Access Management & Governance*
