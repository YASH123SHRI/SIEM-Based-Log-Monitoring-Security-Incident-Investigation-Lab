# Linux Syslog – Collection and Analysis Using Wazuh

## 1. Objective

The objective of this lab is to understand Linux system logs, identify important events recorded through syslog/journald, collect those events using the Wazuh Agent, and investigate them through the Wazuh Dashboard.

Linux system logs can provide valuable information about:

- System services
- Service start/stop events
- System activity
- Kernel messages
- Network-related events
- Scheduled tasks
- Authentication-related activity
- Application behaviour
- System errors
- Configuration changes

In this lab, we will:

- Understand Linux syslog.
- Identify important system log files.
- Examine `/var/log/syslog`.
- Generate controlled system/service events.
- Analyse syslog entries using command-line tools.
- Configure Wazuh to collect syslog.
- Verify that events reach Wazuh.
- Investigate system activity through the Wazuh Dashboard.
- Perform a basic SOC investigation.
- Document findings.

---

# 2. Lab Environment

| Component | Description |
|---|---|
| Linux Endpoint | Ubuntu Linux |
| SIEM | Wazuh |
| Log Collection | Wazuh Agent |
| Primary Log | `/var/log/syslog` |
| Log Analysis | Wazuh Dashboard |
| Additional Tool | `journalctl` |
| Privileges | sudo/root required for some activities |

## Lab Architecture

```text
+-----------------------------+
|        Linux Endpoint       |
|                             |
| Services / Applications     |
| Kernel / System Activity    |
|          |                  |
|          v                  |
|    /var/log/syslog          |
|          |                  |
|          v                  |
|      Wazuh Agent            |
+------------|----------------+
             |
             | Log Collection
             v
+-----------------------------+
|       Wazuh Manager         |
|                             |
| Decoders / Rules            |
| Event Processing            |
+------------|----------------+
             |
             v
+-----------------------------+
|      Wazuh Dashboard        |
|                             |
| Threat Hunting              |
| Event Investigation         |
| Alerts / Monitoring         |
+-----------------------------+
```

---

# 3. Understanding Linux Syslog

Syslog is a standard mechanism used by Linux systems and applications to record events.

On Ubuntu and other Debian-based systems, general system activity is commonly recorded in:

```text
/var/log/syslog
```

Authentication-related events are generally handled separately in:

```text
/var/log/auth.log
```

Therefore:

```text
syslog.md     → General system/service activity
auth-logs.md  → Authentication-related activity
ssh-logs.md   → SSH-specific activity
```

Keeping these topics separate makes the SOC investigation workflow easier to understand.

---

# 4. Common Linux Log Files

| Log File | Purpose |
|---|---|
| `/var/log/syslog` | General system and service activity |
| `/var/log/auth.log` | Authentication and authorization activity |
| `/var/log/kern.log` | Kernel-related messages, where configured |
| `/var/log/dpkg.log` | Package management activity |
| `/var/log/apt/` | APT package management logs |
| `/var/log/boot.log` | Boot-related information, where available |

The exact log files available depend on the Linux distribution and logging configuration.

---

# 5. Check Your Linux Logging Configuration

First identify the Linux distribution:

```bash
cat /etc/os-release
```

Then check whether `/var/log/syslog` exists:

```bash
ls -lh /var/log/syslog
```

If it exists, display the latest entries:

```bash
sudo tail -n 20 /var/log/syslog
```

You can continuously monitor new entries using:

```bash
sudo tail -f /var/log/syslog
```

Stop the command using:

```text
Ctrl + C
```

---

# 6. Understand a Syslog Entry

A typical syslog entry may look similar to:

```text
Nov 13 10:45:21 ubuntu systemd[1]: Started Example Service.
```

Common fields include:

```text
Timestamp
Hostname
Process / Service
PID
Message
```

For example:

```text
Timestamp : Nov 13 10:45:21
Hostname  : ubuntu
Process   : systemd
PID       : 1
Message   : Started Example Service
```

The exact format varies by application and logging configuration.

---

# 7. Practical Lab – Monitor Syslog in Real Time

Open two terminal windows.

### Terminal 1

Run:

```bash
sudo tail -f /var/log/syslog
```

Keep this terminal open.

### Terminal 2

We will generate controlled system activity.

---

# 8. Generate a Controlled Service Event

First identify a service that exists on your machine.

Run:

```bash
systemctl list-units --type=service --state=running | head
```

Choose a non-critical service that you recognize.

For example, if your system has `cron`:

```bash
systemctl status cron
```

Do not stop or disable important services such as:

- NetworkManager
- systemd
- ssh
- systemd-logind
- graphical session services

For this lab, simply inspect the service rather than stopping it.

---

# 9. Generate a Safe Custom Syslog Event

A convenient way to create a controlled syslog event is the `logger` command.

Run:

```bash
logger "SOC-Sentinel Syslog Lab - Test Event"
```

Now return to:

```bash
sudo tail -f /var/log/syslog
```

You should see a new entry containing your message.

This is useful because you can clearly identify the event generated specifically for this lab.

For example:

```text
SOC-Sentinel Syslog Lab - Test Event
```

---

# 10. Generate Different Severity Events

The `logger` command can also specify a facility and severity.

Generate an informational message:

```bash
logger -p user.info "SOC-Sentinel informational test"
```

Generate a warning:

```bash
logger -p user.warning "SOC-Sentinel warning test"
```

Generate an error-level test message:

```bash
logger -p user.err "SOC-Sentinel error test"
```

These are controlled test events.

They do not represent actual system failures.

---

# 11. Search Syslog Using grep

Search for your SOC test events:

```bash
sudo grep "SOC-Sentinel" /var/log/syslog
```

Search for errors:

```bash
sudo grep -i "error" /var/log/syslog | tail -n 20
```

Search for warnings:

```bash
sudo grep -i "warning" /var/log/syslog | tail -n 20
```

Search for service-related messages:

```bash
sudo grep -i "systemd" /var/log/syslog | tail -n 20
```

---

# 12. Analyse Syslog by Process

You can search for messages generated by a particular process.

For example:

```bash
sudo grep "systemd" /var/log/syslog | tail -n 20
```

Or:

```bash
sudo grep "cron" /var/log/syslog | tail -n 20
```

This helps a SOC Analyst identify which system component generated an event.

---

# 13. Analyse Syslog Using journalctl

Modern Linux systems commonly use `systemd-journald`.

View recent system logs:

```bash
sudo journalctl -n 20
```

View logs from the last hour:

```bash
sudo journalctl --since "1 hour ago"
```

View logs from today:

```bash
sudo journalctl --since today
```

Follow new journal events:

```bash
sudo journalctl -f
```

Stop using:

```text
Ctrl + C
```

---

# 14. Compare syslog and journalctl

Run:

```bash
sudo journalctl --since "10 minutes ago"
```

Then:

```bash
sudo tail -n 50 /var/log/syslog
```

Observe that both can contain overlapping system information, depending on the system's logging configuration.

This is useful for understanding the relationship between:

```text
systemd-journald
       |
       v
system logs
       |
       +----> /var/log/syslog
       |
       +----> journalctl
```

The exact relationship depends on the Linux distribution and `rsyslog` configuration.

---

# 15. Configure Wazuh Agent for Syslog Collection

Now configure Wazuh to collect `/var/log/syslog`.

## Step 1: Check Wazuh Agent

Run:

```bash
sudo systemctl status wazuh-agent
```

Expected:

```text
Active: active (running)
```

If necessary:

```bash
sudo systemctl start wazuh-agent
```

---

# 16. Back Up Wazuh Configuration

Before modifying the configuration:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup
```

---

# 17. Add Syslog Collection

Open:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inside the existing `<ossec_config>` section, add:

```xml
<localfile>
  <location>/var/log/syslog</location>
  <log_format>syslog</log_format>
</localfile>
```

Do not create another `<ossec_config>` element.

Do not delete existing Wazuh configuration.

Save the file.

---

# 18. Restart Wazuh Agent

Run:

```bash
sudo systemctl restart wazuh-agent
```

Verify:

```bash
sudo systemctl status wazuh-agent
```

Check:

```bash
sudo systemctl is-active wazuh-agent
```

Expected:

```text
active
```

---

# 19. Check Wazuh Agent Logs

Run:

```bash
sudo tail -n 50 /var/ossec/logs/ossec.log
```

Look for configuration errors or connection problems.

---

# 20. Generate a New Syslog Event for Wazuh

Run:

```bash
logger "SOC-Sentinel Wazuh Syslog Test"
```

Then verify locally:

```bash
sudo grep "SOC-Sentinel Wazuh Syslog Test" /var/log/syslog
```

This confirms that Linux generated the event.

---

# 21. Verify the Event in Wazuh

Open your Wazuh Dashboard.

Navigate to:

```text
Threat Hunting
```

or the equivalent event exploration section in your Wazuh version.

Set an appropriate time range.

Search for your Linux agent.

Look for an event containing:

```text
SOC-Sentinel Wazuh Syslog Test
```

Inspect the event details.

Record:

```text
Agent:
Timestamp:
Location:
Decoder:
Message:
Facility:
Severity:
```

The exact Wazuh field names depend on your Wazuh version and decoder output.

---

# 22. Syslog SOC Investigation Exercise

## Scenario

A Linux server is reporting unusual system activity.

A SOC Analyst receives an alert indicating an unusual system event and needs to determine whether the event represents a legitimate system operation or suspicious activity.

### Investigation Steps

1. Identify the affected Linux host.
2. Identify the timestamp.
3. Identify the process or service responsible.
4. Read the complete log message.
5. Search for related events before and after the event.
6. Check whether the event occurred repeatedly.
7. Check authentication logs for related activity.
8. Check SSH logs if remote access may be involved.
9. Determine whether the activity is expected.
10. Document the investigation.

---

# 23. Correlate Syslog With Authentication Logs

Suppose you find an unusual system event at:

```text
10:35:20
```

Search authentication activity around the same time:

```bash
sudo grep "Nov 13 10:35" /var/log/auth.log
```

Then search syslog:

```bash
sudo grep "Nov 13 10:35" /var/log/syslog
```

The exact date/time format will depend on your system.

The goal is to correlate:

```text
Authentication Event
        |
        v
System Event
        |
        v
Service / Process Activity
        |
        v
SOC Finding
```

---

# 24. SOC Investigation Report Template

```text
Incident Title:
Linux System Event Investigation

Date:
Analyst:

Affected Host:
Wazuh Agent ID:

Log Source:
/var/log/syslog

Event Timestamp:

Process / Service:

Event Message:

Related Events:

1.
2.
3.

Authentication Correlation:

Was there a related authentication event?

Was the activity expected?

Was the activity repeated?

Risk Assessment:

Low / Medium / High / Undetermined

Conclusion:

Recommended Actions:

1.
2.
3.
```

---

# 25. Screenshots for GitHub

Create:

```text
02-Log-Collection/Linux-Logs/screenshots/
```

Recommended screenshots:

| Screenshot | Description |
|---|---|
| `01-syslog-output.png` | `/var/log/syslog` output |
| `02-syslog-live-monitoring.png` | `tail -f /var/log/syslog` |
| `03-logger-test-event.png` | Controlled test event |
| `04-journalctl-output.png` | `journalctl` output |
| `05-wazuh-syslog-config.png` | Wazuh Agent configuration |
| `06-wazuh-agent-status.png` | Active Wazuh Agent |
| `07-wazuh-syslog-event.png` | Syslog event visible in Wazuh |
| `08-wazuh-event-details.png` | Expanded event details |
| `09-syslog-investigation.png` | Investigation evidence |

Review screenshots before publishing them.

Do not expose:

- Passwords
- API keys
- Tokens
- Personal information
- Unnecessary internal network information

---

# 26. Key Takeaways

After completing this lab, you should understand:

- What Linux syslog is.
- The purpose of `/var/log/syslog`.
- How to monitor system events in real time.
- How to generate controlled test events using `logger`.
- How to search syslog using `grep`.
- How to use `journalctl`.
- How to configure Wazuh to collect syslog.
- How to verify syslog events in Wazuh.
- How to correlate system events with authentication activity.
- How a SOC Analyst investigates Linux system activity.

---

# 27. References

### Linux

- `journalctl` documentation:
  https://man7.org/linux/man-pages/man1/journalctl.1.html

- `logger` documentation:
  https://man7.org/linux/man-pages/man1/logger.1.html

### Wazuh

- Wazuh Log Data Collection:
  https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/index.html

- Wazuh `localfile` Configuration:
  https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html

---
**Log Source:** `/var/log/syslog`

**SIEM:** Wazuh

**Status:** In Progress – Update after completing the practical lab.
