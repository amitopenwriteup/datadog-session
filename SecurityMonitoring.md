# Security Monitoring and Compliance Workshop
### A 2-Hour Hands-On Training Guide

---

> **Duration:** 2 Hours  
> **Format:** Instructor-Led Workshop  
> **Level:** Intermediate  
> **Prerequisites:** Basic understanding of IT infrastructure and networking concepts

---

## Workshop Agenda

| # | Module | Duration |
|---|--------|----------|
| 1 | Introduction & Goals | 10 min |
| 2 | Security Monitoring Overview | 30 min |
| 3 | Threat Detection and Alerts | 40 min |
| 4 | Hands-On Lab Exercises | 25 min |
| 5 | Review, Q&A & Wrap-Up | 15 min |

---

## Module 1 — Introduction & Goals
**Duration: 10 minutes**

### Workshop Objectives

By the end of this workshop, participants will be able to:

1. Explain the purpose and components of a security monitoring framework.
2. Identify key data sources used for monitoring infrastructure and applications.
3. Describe how threat detection works and what triggers security alerts.
4. Understand alert severity levels and the proper response workflow.
5. Apply basic triage techniques to evaluate and prioritize security events.

### Trainer Instructions

**Step 1:** Welcome participants and introduce yourself and your role.

**Step 2:** Ask the room the following icebreaker question and collect 3–4 verbal responses:

> *"Has anyone here ever received a security alert at work? What happened next?"*

**Step 3:** Display the agenda and explain the two-part structure:
- Part A: Conceptual (Security Monitoring Overview)
- Part B: Applied (Threat Detection and Alerts)

**Step 4:** Set ground rules for the session:
- Questions are welcome at any time.
- Exercises are individual unless otherwise stated.
- All lab scenarios are fictional and for training purposes only.

**Step 5:** Confirm all participants have the workshop handout or have access to this document.

---

## Module 2 — Security Monitoring Overview
**Duration: 30 minutes**

---

### Section 2.1 — What Is Security Monitoring?
**Time: 10 minutes**

Security monitoring is the continuous process of collecting, analyzing, and acting on data from across an organization's IT environment to detect and respond to threats in real time.

**Step 1:** Define security monitoring to the group using the following points:

- It is **proactive**, not just reactive.
- It covers **networks, endpoints, applications, cloud infrastructure, and users**.
- The goal is to reduce the **mean time to detect (MTTD)** and **mean time to respond (MTTR)** to incidents.

**Step 2:** Introduce the three pillars of security monitoring:

```
┌─────────────────────────────────────────────┐
│         THREE PILLARS OF MONITORING         │
│                                             │
│   1. VISIBILITY   → Know what you have      │
│   2. DETECTION    → Know what is wrong      │
│   3. RESPONSE     → Know what to do         │
└─────────────────────────────────────────────┘
```

**Step 3:** Ask participants: *"What do you think is the biggest blind spot in security monitoring for most organizations?"* Allow 2 minutes of open discussion.

**Step 4:** Transition to the key components section.

---

### Section 2.2 — Key Components of a Security Monitoring Program
**Time: 10 minutes**

**Step 1:** Walk through each component listed below. Spend approximately 1.5 minutes per item.

#### 1. Security Information and Event Management (SIEM)
- Centralized platform that collects and correlates log data from multiple sources.
- Generates alerts based on predefined rules and behavioral baselines.
- Examples: Splunk, Microsoft Sentinel, IBM QRadar.

#### 2. Log Management
- Systematic collection and storage of log data from:
  - Firewalls and network devices
  - Operating systems (Windows Event Logs, Linux syslog)
  - Applications and databases
  - Cloud platforms (AWS CloudTrail, Azure Monitor)
- Logs must be **timestamped, tamper-proof, and retained** per compliance policy.

#### 3. Network Traffic Analysis (NTA)
- Monitors packet flows and detects anomalies in network behavior.
- Identifies lateral movement, data exfiltration, and unusual port activity.

#### 4. Endpoint Detection and Response (EDR)
- Agent-based monitoring on individual devices.
- Detects malware execution, suspicious process chains, and privilege escalation.

#### 5. User and Entity Behavior Analytics (UEBA)
- Establishes behavioral baselines for users and systems.
- Flags deviations such as logins at unusual hours or bulk file access.

**Step 2:** Draw or display the following data flow diagram on the board:

```
Data Sources → Log Collectors → SIEM → Correlation Engine → Alerts → SOC Analyst
     ↑                                        ↑
[Endpoints, Network,                   [Rules + Baselines]
 Cloud, Applications]
```

**Step 3:** Confirm understanding by asking: *"Which component would you use to detect a user logging in at 3 AM from a foreign country?"* (Answer: UEBA or SIEM with geo-based rule.)

---

### Section 2.3 — Compliance and Monitoring Frameworks
**Time: 10 minutes**

**Step 1:** Explain why compliance drives monitoring requirements.

Organizations must monitor their environments not just for security reasons, but because **regulatory frameworks mandate it**. Non-compliance can result in fines, legal liability, and loss of business.

**Step 2:** Review the following compliance frameworks and their monitoring requirements:

| Framework | Industry | Key Monitoring Requirement |
|-----------|----------|---------------------------|
| **PCI-DSS** | Payment / Retail | Log all access to cardholder data; retain 12 months |
| **HIPAA** | Healthcare | Audit access to Protected Health Information (PHI) |
| **SOC 2** | Technology / SaaS | Monitor for unauthorized access and system availability |
| **ISO 27001** | All industries | Implement event logging and operational monitoring |
| **NIST CSF** | Government / General | Detect anomalies; respond and recover from incidents |
| **GDPR** | EU / Global | Monitor and report data breaches within 72 hours |

**Step 3:** Emphasize the key principle:

> *"Compliance is the floor, not the ceiling. Meeting a regulatory standard does not guarantee security — it guarantees a minimum level of accountability."*

**Step 4:** Distribute or reference the **Compliance Monitoring Checklist** (Appendix A at the end of this document).

**Step 5:** Take 2 minutes of questions before moving to Module 3.

---

## Module 3 — Threat Detection and Alerts
**Duration: 40 minutes**

---

### Section 3.1 — How Threat Detection Works
**Time: 12 minutes**

**Step 1:** Introduce the two primary detection models:

#### Model A: Signature-Based Detection
- Compares activity against a **known library of attack patterns** (signatures).
- Fast and accurate for **known threats**.
- Weakness: Completely blind to **new, zero-day, or obfuscated attacks**.

#### Model B: Behavioral / Anomaly-Based Detection
- Learns what "normal" looks like and flags **deviations**.
- Effective against unknown and insider threats.
- Weakness: Can generate **false positives** until baselines are tuned.

**Step 2:** Explain how modern detection combines both models:

```
Incoming Event
      │
      ├──► Signature Match? ──► YES ──► Immediate Alert (HIGH confidence)
      │
      └──► Behavioral Baseline Check ──► Deviation? ──► Risk Scored Alert
```

**Step 3:** Introduce the **MITRE ATT&CK Framework** as a reference for understanding attacker behavior:

- A globally recognized matrix of **tactics and techniques** used by real-world threat actors.
- Organized into 14 tactical categories:
  1. Reconnaissance
  2. Resource Development
  3. Initial Access
  4. Execution
  5. Persistence
  6. Privilege Escalation
  7. Defense Evasion
  8. Credential Access
  9. Discovery
  10. Lateral Movement
  11. Collection
  12. Command and Control
  13. Exfiltration
  14. Impact

**Step 4:** Show one example scenario mapped to MITRE ATT&CK:

> **Scenario:** An attacker sends a phishing email with a malicious attachment.
>
> - **Initial Access** → Spearphishing Attachment (T1566.001)
> - **Execution** → User Opens Attachment → Macro Runs (T1059.005)
> - **Persistence** → Registry Run Key Added (T1547.001)

**Step 5:** Ask participants: *"Where in the MITRE ATT&CK chain is detection most likely to succeed in your organization?"*

---

### Section 3.2 — Alert Severity Levels and Classification
**Time: 10 minutes**

**Step 1:** Explain the standard alert severity taxonomy used across most SIEM platforms:

| Severity Level | Color Code | Definition | Response Time |
|---------------|------------|------------|---------------|
| **Critical** | 🔴 Red | Active breach or confirmed threat in progress | Immediate (< 15 min) |
| **High** | 🟠 Orange | Strong indicators of compromise; likely malicious | < 1 hour |
| **Medium** | 🟡 Yellow | Suspicious activity requiring investigation | < 4 hours |
| **Low** | 🔵 Blue | Policy violations or minor anomalies | < 24 hours |
| **Informational** | ⚪ White | Audit events; no immediate threat | Reviewed in reports |

**Step 2:** Explain **alert fatigue**:

> *"When analysts receive hundreds or thousands of low-quality alerts per day, they become desensitized. Critical alerts get buried. This is one of the leading causes of breach escalation."*

**Step 3:** Introduce the three solutions to alert fatigue:

1. **Tuning** — Refine rules to reduce false positives.
2. **Prioritization** — Use risk scoring to surface the most important alerts.
3. **Suppression** — Temporarily silence known-safe events from trusted sources.

**Step 4:** Walk through this example of a poorly tuned rule vs. a well-tuned rule:

```
❌ Bad Rule:
"Alert every time a user logs in from a new device."
→ Result: 500 alerts/day, mostly legitimate.

✅ Better Rule:
"Alert when a user logs in from a new device AND a different country
 AND at a time outside their normal work hours."
→ Result: 5–10 targeted, high-confidence alerts/day.
```

---

### Section 3.3 — The Alert Response Workflow
**Time: 10 minutes**

**Step 1:** Introduce the 6-step alert triage process. Walk through each step carefully.

#### Step 1 — Receive & Acknowledge
- The alert is assigned to an analyst.
- Analyst acknowledges receipt in the ticketing system.
- Clock starts on SLA response time.

#### Step 2 — Validate (True Positive or False Positive?)
- Review the raw event data behind the alert.
- Ask: *Does this activity make sense in context?*
- Common sources for validation: log details, asset owner confirmation, change records.

#### Step 3 — Classify
- Assign the alert to a category:
  - Malware infection
  - Unauthorized access
  - Data exfiltration attempt
  - Policy violation
  - Misuse of credentials

#### Step 4 — Scope
- Determine the blast radius:
  - How many systems are affected?
  - Is the threat still active or contained?
  - Are user accounts compromised?

#### Step 5 — Contain
- Take immediate containment action if necessary:
  - Isolate the endpoint from the network.
  - Disable the compromised user account.
  - Block the offending IP address at the firewall.

#### Step 6 — Document and Escalate
- Record all findings in the incident ticket.
- Escalate to Tier 2 / Incident Response team if threat is confirmed.
- Notify stakeholders as per escalation policy.

**Step 2:** Display the workflow as a flowchart:

```
Alert Generated
      │
      ▼
Analyst Acknowledges
      │
      ▼
Is it a True Positive?
  ├── NO  ──► Mark as False Positive → Tune Rule → Close Ticket
  └── YES ──► Classify → Scope → Contain → Escalate → Document
```

---

### Section 3.4 — Common Threat Scenarios and Indicators of Compromise (IOCs)
**Time: 8 minutes**

**Step 1:** Define Indicators of Compromise (IOCs):

> *An IOC is a piece of forensic data that suggests a system may have been breached or is actively under attack.*

**Step 2:** Walk through the following common IOCs and their associated threat scenarios:

| IOC Type | Example | Associated Threat |
|----------|---------|-------------------|
| **Unusual login location** | Login from Russia when user is in India | Account takeover |
| **High outbound data volume** | 10GB transferred at 2 AM | Data exfiltration |
| **Unknown process execution** | `cmd.exe` spawned by `Word.exe` | Macro malware |
| **Multiple failed logins** | 200 attempts in 60 seconds | Brute-force attack |
| **New admin account created** | After business hours, no change request | Privilege escalation |
| **Suspicious DNS queries** | Regular beaconing to random-looking domains | Command & Control (C2) |
| **Disabled security tools** | Antivirus stopped on a workstation | Defense evasion |

**Step 3:** Ask each participant to identify which IOC they consider most dangerous in their environment and why. Allow 2 minutes of quick discussion.

---

## Module 4 — Hands-On Lab Exercises
**Duration: 25 minutes**

---

### Lab 1 — Alert Triage Simulation
**Time: 10 minutes | Individual Exercise**

**Instructions:**

**Step 1:** Read the following three alert scenarios carefully.

**Step 2:** For each alert, answer the four questions below.

**Step 3:** Be prepared to share your answers with the group.

---

**Alert A — Severity: HIGH**
> User `jsmith@company.com` logged in successfully from IP `185.220.101.47` (Tor exit node) at 03:14 AM. The user's normal working hours are 9 AM – 6 PM. The last normal login was 8 hours ago from the corporate office IP.

**Alert B — Severity: MEDIUM**
> Antivirus detected and quarantined `Trojan.GenericKD.45` on workstation `DESKTOP-HQ922`. The file was located in `C:\Users\Public\Downloads\invoice_march.exe`. The endpoint is still connected to the network.

**Alert C — Severity: LOW**
> User `agarcia@company.com` accessed 45 files in the HR shared drive within 10 minutes. The user is an HR Manager. No data was transferred externally.

---

**Questions to Answer for Each Alert:**

1. Is this alert a true positive or a likely false positive? Why?
2. What is your immediate containment action (if any)?
3. What additional information would you request to complete your investigation?
4. Who would you escalate this to, and at what point?

---

### Lab 2 — Rule Writing Exercise
**Time: 8 minutes | Pair Exercise**

**Instructions:**

**Step 1:** Pair up with the person next to you.

**Step 2:** Read the following threat scenario:

> *Your organization has seen a recent wave of ransomware attacks targeting companies in your industry. Attackers are using phishing emails to deliver malicious PowerShell scripts that encrypt files and disable backups.*

**Step 3:** Together, write **two detection rules** in plain English that a SIEM could use to detect this attack pattern. Use the format:

```
RULE NAME: [Your Rule Name]
TRIGGER: Alert when [specific event or combination of events]
CONDITIONS: [Additional context or thresholds]
SEVERITY: [Critical / High / Medium / Low]
```

**Step 4:** After 5 minutes, each pair shares one of their rules with the group.

---

### Lab 3 — Compliance Gap Identification
**Time: 7 minutes | Group Discussion**

**Instructions:**

**Step 1:** As a group, review the following monitoring scenario:

> *A mid-sized e-commerce company processes credit card payments and stores customer PII. They have a basic firewall, antivirus on all endpoints, and send logs from their web servers to a central server. Logs are kept for 30 days. They have no SIEM, no alerting on failed logins, and no log review process.*

**Step 2:** Identify at least **four compliance gaps** in this setup.

**Step 3:** For each gap, name:
- Which framework it violates (PCI-DSS, GDPR, HIPAA, etc.)
- What the organization should implement to remediate it

**Step 4:** The trainer records answers on the whiteboard and facilitates brief discussion.

---

## Module 5 — Review, Q&A & Wrap-Up
**Duration: 15 minutes**

---

### Section 5.1 — Key Takeaways Review
**Time: 7 minutes**

**Step 1:** Recap the core concepts from each module by asking participants to supply the answers:

- *"What are the three pillars of security monitoring?"*
  > Visibility, Detection, Response

- *"What is the difference between signature-based and behavioral detection?"*
  > Signatures catch known threats; behavioral catches unknown/anomalous behavior.

- *"What are the 6 steps in the alert triage process?"*
  > Receive, Validate, Classify, Scope, Contain, Document/Escalate

- *"Name one compliance framework and its key monitoring requirement."*
  > (Accept any valid answer from Section 2.3 table.)

- *"What is alert fatigue and how do you reduce it?"*
  > Tuning, prioritization, and suppression.

**Step 2:** Share the following key principles as final takeaways:

```
✔ Monitor continuously — threats don't keep business hours.
✔ Log everything — you can't investigate what you can't see.
✔ Tune your rules — quality over quantity in alerting.
✔ Comply, but don't stop there — compliance is the minimum standard.
✔ Document everything — investigations depend on a clear record.
```

---

### Section 5.2 — Open Q&A
**Time: 5 minutes**

**Step 1:** Open the floor for questions. Prompt with:

> *"What was the most surprising thing you learned today?"*
> *"What would you go back and change in your current monitoring setup?"*

**Step 2:** Answer questions as time allows. Park unanswered questions in a visible list and commit to follow-up by email.

---

### Section 5.3 — Post-Workshop Actions
**Time: 3 minutes**

**Step 1:** Ask each participant to commit to **one action item** they will take in the next 30 days based on today's workshop.

**Step 2:** Distribute or point participants to the following resources:

- **MITRE ATT&CK Framework:** https://attack.mitre.org
- **NIST Cybersecurity Framework:** https://www.nist.gov/cyberframework
- **SANS Reading Room (Free Papers):** https://www.sans.org/reading-room
- **CIS Controls:** https://www.cisecurity.org/controls

**Step 3:** Distribute the post-workshop feedback form (paper or digital link).

**Step 4:** Thank participants for their time and close the session.

---

## Appendix A — Compliance Monitoring Checklist

Use this checklist to assess the current state of your organization's monitoring program.

### Log Collection

- [ ] Logs collected from all firewalls and network devices
- [ ] Logs collected from all servers (Windows Event Logs, syslog)
- [ ] Logs collected from all cloud platforms (AWS, Azure, GCP)
- [ ] Logs collected from all critical business applications
- [ ] Log integrity verified (tamper-evidence enabled)
- [ ] Log retention policy defined and enforced (minimum 12 months for PCI-DSS)

### Alerting and Detection

- [ ] SIEM deployed and operational
- [ ] Alerting enabled for failed login attempts (threshold-based)
- [ ] Alerting enabled for privileged account usage (admin logins, sudo)
- [ ] Alerting enabled for large data transfers or exfiltration attempts
- [ ] Alerting enabled for after-hours access to sensitive systems
- [ ] Rules reviewed and tuned in the last 90 days

### Response Readiness

- [ ] Documented incident response plan exists
- [ ] Alert severity levels and SLAs defined
- [ ] On-call escalation contacts documented and current
- [ ] Incident ticketing system in use
- [ ] Post-incident review process defined

### Compliance-Specific

- [ ] PCI-DSS: Access to cardholder data monitored and logged
- [ ] HIPAA: Audit logs for all PHI access reviewed regularly
- [ ] GDPR: Process in place to detect and report breaches within 72 hours
- [ ] SOC 2: Evidence of continuous monitoring collected for auditors

---

## Appendix B — Glossary of Key Terms

| Term | Definition |
|------|-----------|
| **Alert Fatigue** | Desensitization of analysts due to high volumes of low-quality alerts |
| **Baseline** | A representation of normal behavior used for anomaly detection |
| **CSIRT** | Computer Security Incident Response Team |
| **EDR** | Endpoint Detection and Response |
| **False Positive** | An alert triggered by benign activity incorrectly identified as a threat |
| **IOC** | Indicator of Compromise — a data artifact suggesting a breach |
| **MITRE ATT&CK** | A knowledge base of adversary tactics and techniques |
| **MTTD** | Mean Time to Detect — average time to identify a threat |
| **MTTR** | Mean Time to Respond — average time to contain and resolve a threat |
| **NTA** | Network Traffic Analysis |
| **SIEM** | Security Information and Event Management |
| **SOC** | Security Operations Center |
| **True Positive** | An alert that correctly identifies a real threat |
| **UEBA** | User and Entity Behavior Analytics |

---

*Workshop developed for internal security training. All lab scenarios are fictional and used for educational purposes only.*
