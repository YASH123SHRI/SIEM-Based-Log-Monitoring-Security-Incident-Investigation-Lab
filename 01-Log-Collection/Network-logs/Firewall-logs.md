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
                │                         │
                │          UFW            │
                │           │             │
                │           ▼             │
                │      /var/log/ufw.log   │
                └─────────────────────────┘
```
---

# 3. EVENT

Network Traffic & Connections

Event ID Description	When to Use It

5156	Permitted Connection	Tracks when the firewall allows an application or port to connect.

5157	Blocked Connection	Tracks when the firewall blocks an inbound or outbound connection.

5152	Blocked Packet	Logs individual dropped packets (more granular than a full connection block).

5154	Permitted Listen	Logs when an application is allowed to listen on a port for incoming connections.

5155	Blocked Listen	Logs when an application is blocked from listening on a port.

5158	Permitted Port Bind	Logs when an application is allowed to bind to a local port.

5159	Blocked Port Bind	Logs when an application is blocked from binding to a local port.

5031	Application Blocked	Logs when Windows Firewall specifically blocks an app from accepting incoming connections.


Firewall Rule & Policy Changes

If you are auditing who or what is altering your firewall configuration, filter for these IDs (documented in the Microsoft Learn Security Auditing Guide):
• 4946 — A change was made to Windows Firewall exception list. A rule was added.
• 4947 — A change was made to Windows Firewall exception list. A rule was modified.
• 4948 — A change was made to Windows Firewall exception list. A rule was deleted.
• 4950 — A Windows Firewall setting was changed (such as turning a profile on or off).
• 4954 — Windows Firewall Group Policy settings changed, and the new settings were applied. 

Firewall Service & Driver Status

These IDs help troubleshoot if the firewall service itself crashes or stops running:
• 5024 — The Windows Firewall Service started successfully.
• 5025 — The Windows Firewall Service was stopped.
• 5030 — The Windows Firewall Service failed to start.
• 5037 — The Windows Firewall Driver detected a critical runtime error and terminated.

---
# 4. Screenshot

<img width="1312" height="302" alt="image" src="https://github.com/user-attachments/assets/ff1514f2-aced-4aa8-bb89-b0c9d7ec09e0" />
