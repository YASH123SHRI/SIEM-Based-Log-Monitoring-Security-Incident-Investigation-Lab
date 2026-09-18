# SIEM-Based-Log-Monitoring-Security-Incident-Investigation-Lab

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
│   │   ├── incident-summary.md
│   │   ├── timeline.md
│   │   ├── evidence.md
│   │   ├── analysis.md
│   │   └── recommendations.md
│   │
│   ├── Case-02-Suspicious-PowerShell
│   │   ├── incident-summary.md
│   │   ├── timeline.md
│   │   ├── evidence.md
│   │   ├── analysis.md
│   │   └── recommendations.md
│   │
│   ├── Case-03-SSH-Brute-Force
│   │   └── ...
│   │
│   └── Case-04-Port-Scanning
│       └── ...
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
