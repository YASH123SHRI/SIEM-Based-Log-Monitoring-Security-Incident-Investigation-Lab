# SOC Sentinel: SIEM-Based Log Monitoring & Security Incident Investigation Lab

A hands-on **Security Operations Center (SOC) laboratory** focused on SIEM deployment, centralized log monitoring, security event analysis, detection engineering, alert triage, incident investigation, IOC analysis, and basic incident response.

This project is designed to simulate the workflow of a **SOC Level 1 Analyst** working with Windows, Linux, and network security logs.

---

## 🎯 Project Objective

The primary objective of this lab is to understand and demonstrate how a SOC analyst:

```text
Collects Logs
      ↓
Monitors Security Events
      ↓
Analyzes Logs
      ↓
Identifies Suspicious Activity
      ↓
Generates / Investigates Alerts
      ↓
Performs Alert Triage
      ↓
Investigates the Incident
      ↓
Analyzes IOCs
      ↓
Recommends Response Actions
      ↓
Documents the Incident
```

The project focuses on practical SOC operations rather than only theoretical SIEM concepts.

---

## Lab Architecture

```text
                    ┌──────────────────────┐
                    │      Kali Linux      │
                    │  Security Testing /  │
                    │  Attack Simulation   │
                    └──────────┬───────────┘
                               │
                               │ Security Events
                               ▼
             ┌──────────────────────────────────┐
             │          Monitored Hosts         │
             │                                  │
             │  ┌────────────┐  ┌────────────┐ │
             │  │ Windows    │  │   Linux    │ │
             │  │    Host    │  │     VM     │ │
             │  └─────┬──────┘  └─────┬──────┘ │
             │        │               │        │
             │ Windows Logs       Linux Logs   │
             │ Security           Auth         │
             │ System             Syslog       │
             │ PowerShell         SSH          │
             └────────┬───────────────┬─────────┘
                      │               │
                      └───────┬───────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │        SIEM         │
                    │       Splunk        │
                    ├─────────────────────┤
                    │ Log Collection       │
                    │ Searching            │
                    │ Analysis             │
                    │ Detection             │
                    │ Alerting              │
                    │ Dashboards            │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     SOC Analyst     │
                    ├─────────────────────┤
                    │ Alert Triage         │
                    │ Investigation        │
                    │ IOC Analysis         │
                    │ Incident Response    │
                    │ Documentation        │
                    └─────────────────────┘
```

---

# Technologies & Tools

| Category          | Technologies / Tools                         |
| ----------------- | -------------------------------------------- |
| SIEM              | Splunk                                       |
| Operating Systems | Windows 11, Linux                            |
| Security Testing  | Kali Linux                                   |
| Virtualization    | VirtualBox                                   |
| Log Sources       | Windows Event Logs, Linux Logs, Network Logs |
| Query Language    | Splunk SPL                                   |
| Network Analysis  | Wireshark                                    |
| IOC Investigation | VirusTotal                                   |
| Documentation     | Markdown                                     |
| Evidence          | Screenshots, Logs, SIEM Events               |

---

# Log Sources

This project works with multiple types of security telemetry.

## Windows Logs

The lab focuses on:

* Windows Security Events
* Windows System Events
* PowerShell Logs
* Authentication Events
* Account-related Events
* Process-related Events

Examples of security events investigated include:

```text
4624 - Successful Logon
4625 - Failed Logon
```

Additional Windows events will be analyzed where relevant to the investigation.

---

## Linux Logs

Linux telemetry includes:

* Authentication logs
* SSH events
* Syslog
* User authentication activity
* System activity

These logs are used to investigate suspicious authentication behavior and other security events.

---

## Network Logs

Network-related telemetry includes:

* Firewall events
* Network connection activity
* Source and destination information
* Port activity
* Suspicious network behavior

---

# Log Analysis

The log analysis phase focuses on understanding how individual events can provide evidence of suspicious activity.

### Areas Covered

* Windows Event Analysis
* Linux Authentication Analysis
* Failed Login Analysis
* Successful Login Analysis
* Suspicious Process Analysis
* Source IP analysis
* User/account analysis
* Timestamp analysis
* Event correlation

The objective is not simply to read logs, but to determine:

> **What happened, when it happened, which system/user was involved, and whether the activity requires further investigation.**

---

# Detection Rules

Detection logic is created to identify common security scenarios relevant to SOC L1 monitoring.

Planned detections include:

* Brute-force activity
* Multiple failed login attempts
* Suspicious login activity
* Privilege-related activity
* Suspicious PowerShell execution
* Network port scanning
* Authentication anomalies

Each detection will document:

```text
Detection Name
        ↓
Data Source
        ↓
Detection Logic
        ↓
Trigger Condition
        ↓
Severity
        ↓
Potential False Positives
        ↓
Investigation Steps
        ↓
Recommended Response
```

---

# SIEM Dashboards

The project includes dashboards designed to provide a quick overview of security activity.

### Authentication Dashboard

Monitors:

* Successful logins
* Failed logins
* Authentication trends
* Top users
* Source IP activity

### Failed Login Dashboard

Monitors:

* Failed authentication attempts
* Repeated failures
* Source IPs
* Target accounts
* Activity over time

### Network Monitoring Dashboard

Monitors:

* Network activity
* Source IPs
* Destination ports
* Connection attempts
* Suspicious network behavior

### Suspicious Activity Dashboard

Provides a consolidated view of potentially suspicious events and alerts.

---

# Security Investigations

The investigation phase simulates common SOC L1 cases.

## Case 01 — Brute-Force Investigation

Investigation of repeated authentication failures to determine:

* Source IP
* Target account
* Number of attempts
* Time window
* Authentication outcome
* Related activity

---

## Case 02 — Suspicious PowerShell Activity

Investigation of potentially suspicious PowerShell execution.

The investigation focuses on:

* User
* Host
* Timestamp
* PowerShell event information
* Available command/process evidence
* Related security events
* Whether additional investigation is required

---

## Case 03 — SSH Brute-Force Investigation

Analysis of repeated SSH authentication attempts against a Linux system.

Investigation includes:

* Source IP
* Target account
* Authentication failures
* Successful authentication
* Timeline
* Related activity

---

## Case 04 — Port Scanning Investigation

Investigation of simulated network reconnaissance activity.

Analysis includes:

* Source host
* Target host
* Ports targeted
* Number of connection attempts
* Time window
* Network behavior

---

# Incident Response

The project follows a basic incident-response workflow:

```text
Alert
  ↓
Triage
  ↓
Investigation
  ↓
Containment
  ↓
Eradication
  ↓
Recovery
  ↓
Lessons Learned
```

The response documentation focuses on what a SOC analyst should do after identifying potentially malicious activity.

---

# IOC Analysis

Potential Indicators of Compromise are investigated where applicable.

### IP Address

* Source/destination IP
* Reputation
* Associated activity

### Domain

* Domain information
* Reputation
* Related indicators

### Hash

* File hash
* Reputation
* Malware intelligence

### VirusTotal

VirusTotal is used as an external intelligence source for controlled lab indicators where appropriate.

---

# SOC Incident Reports

Each investigation is documented in a structured SOC report.

Example structure:

```text
Incident ID
Incident Summary
Detection Source
Severity
Affected Asset
User
Timeline
Evidence
Investigation
Indicators of Compromise
Analysis
False Positive Assessment
Recommended Response
Conclusion
```

The purpose is to practice communicating technical findings in a format suitable for security operations documentation.

---

# Project Structure

```text
SOC-Sentinel-SIEM-Log-Monitoring
│
├── README.md
│
├── 01-SIEM-Deployment
│   ├── siem-installation.md
│   ├── configuration.md
│   ├── data-sources.md
│   └── screenshots/
│
├── 02-Log-Collection
│   ├── Windows-Logs
│   │   ├── security-events.md
│   │   ├── system-events.md
│   │   └── powershell-logs.md
│   │
│   ├── Linux-Logs
│   │   ├── auth-logs.md
│   │   ├── syslog.md
│   │   └── ssh-logs.md
│   │
│   └── Network-Logs
│       ├── firewall-logs.md
│       └── network-events.md
│
├── 03-Log-Analysis
│   ├── Windows-Event-Analysis.md
│   ├── Linux-Authentication-Analysis.md
│   ├── Failed-Login-Analysis.md
│   ├── Successful-Login-Analysis.md
│   └── Suspicious-Process-Analysis.md
│
├── 04-Detection-Rules
│   ├── brute-force-detection.md
│   ├── suspicious-login.md
│   ├── privilege-escalation.md
│   ├── suspicious-powershell.md
│   ├── port-scan-detection.md
│   └── multiple-failed-logins.md
│
├── 05-SIEM-Dashboards
│   ├── authentication-dashboard.png
│   ├── failed-login-dashboard.png
│   ├── network-monitoring-dashboard.png
│   ├── suspicious-activity-dashboard.png
│   └── dashboard-documentation.md
│
├── 06-Security-Investigations
│   ├── Case-01-Brute-Force
│   ├── Case-02-Suspicious-PowerShell
│   ├── Case-03-SSH-Brute-Force
│   └── Case-04-Port-Scanning
│
├── 07-Incident-Response
│   ├── triage-process.md
│   ├── containment.md
│   ├── eradication.md
│   ├── recovery.md
│   └── incident-response-checklist.md
│
├── 08-IOC-Analysis
│   ├── ip-analysis.md
│   ├── domain-analysis.md
│   ├── hash-analysis.md
│   └── virustotal-investigation.md
│
├── 09-SOC-Reports
│   ├── incident-report-01.md
│   ├── incident-report-02.md
│   ├── incident-report-03.md
│   └── final-soc-report.pdf
│
└── 10-Screenshots
    ├── SIEM
    ├── Alerts
    ├── Logs
    ├── Dashboards
    └── Investigations
```

---

# Skills Demonstrated

This project demonstrates practical exposure to:

* SIEM
* Splunk
* Splunk SPL
* Log monitoring
* Log analysis
* Windows Event Logs
* Linux authentication logs
* SSH monitoring
* PowerShell logging
* Network monitoring
* Alert investigation
* Alert triage
* Detection rules
* Security incident investigation
* IOC analysis
* Incident response
* Security documentation
* Basic threat detection
* Security event correlation

---

# SOC L1 Workflow Demonstrated

```text
             ┌──────────────────┐
             │   LOG SOURCES    │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │  SIEM / SPLUNK   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │ LOG MONITORING   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │ DETECTION RULE   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │      ALERT       │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │   ALERT TRIAGE   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │  INVESTIGATION   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │   IOC ANALYSIS   │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │ RESPONSE ACTION  │
             └────────┬─────────┘
                      ↓
             ┌──────────────────┐
             │ SOC REPORTING    │
             └──────────────────┘
```

---

# Disclaimer

All security testing and attack simulations in this project are performed within an isolated, controlled laboratory environment for educational and defensive security purposes.

No unauthorized systems, networks, accounts, or data are targeted.

---

# Author

**Yash Shrivastava**
**Focus:** Cybersecurity | SOC Analyst | SIEM | Security Monitoring | Incident Investigation

---

## 📌 Project Status

**Status:** 🚧 In Progress

This repository will be updated as additional SIEM configurations, log sources, detection rules, investigations, dashboards, and SOC reports are completed.
