# Firewall Logs Collection & Analysis

## 1. Objective

The objective of this lab is to understand how firewall logs are generated, collected, forwarded to **Wazuh**, and analyzed for suspicious network activity.

In this lab, we will:

- Understand basic firewall logging concepts
- Generate controlled firewall events on a Linux system
- Examine firewall logs locally
- Configure the Wazuh Agent to collect firewall logs
- Verify the logs in the Wazuh Dashboard
- Investigate blocked/allowed network connections
- Identify repeated connection attempts
- Correlate firewall events with other security logs

---

# 2. Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Wazuh Manager | Linux |
| Wazuh Dashboard | Web Interface |
| Firewall | UFW |
| Endpoint | Ubuntu/Debian Linux |
| Log Location | `/var/log/ufw.log` |
| Log Format | Syslog |
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
                │      Ubuntu Endpoint    │
                │                          │
                │          UFW             │
                │           │              │
                │           ▼              │
                │      /var/log/ufw.log    │
                └──────────────────────────┘
