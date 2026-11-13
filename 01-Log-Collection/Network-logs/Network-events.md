# Network Events Collection & Analysis

## 1. Objective

The objective of this lab is to understand how network events can be collected from a Linux endpoint, forwarded to **Wazuh**, and analyzed from a SOC Analyst perspective.

In this lab, we will:

- Understand basic network events
- Monitor active network connections
- Identify listening ports and services
- Generate controlled network activity
- Capture network connection information
- Collect network-related logs using Wazuh
- Analyze suspicious or unexpected network activity
- Correlate network events with firewall and authentication logs
- Document findings as a SOC investigation

---

# 2. Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Wazuh Manager | Linux |
| Wazuh Dashboard | Web Interface |
| Endpoint | Ubuntu/Debian Linux |
| Network Tools | `ss`, `ip`, `ping`, `curl`, `nc` |
| Firewall | UFW |
| Primary Logs | `/var/log/syslog`, `/var/log/ufw.log` |
| Analysis | Wazuh + Linux CLI |

### Example Architecture

```text
                   ┌─────────────────────────┐
                   │     Wazuh Dashboard     │
                   │      Threat Hunting     │
                   └────────────┬────────────┘
                                │
                                │
                   ┌────────────▼────────────┐
                   │      Wazuh Manager      │
                   └────────────┬────────────┘
                                │
                           Wazuh Agent
                                │
                   ┌────────────▼────────────┐
                   │      Linux Endpoint     │
                   │                         │
                   │  Network Connections    │
                   │  Listening Services     │
                   │  Firewall Events        │
                   │  System Events          │
                   └─────────────────────────┘
