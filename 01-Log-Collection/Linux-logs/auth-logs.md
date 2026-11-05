# Linux Authentication Logs – Collection and Analysis Using Wazuh

## 1. Objective

The objective of this lab is to understand Linux authentication logs, collect authentication-related events using the Wazuh Agent, and investigate those events through the Wazuh Dashboard.

Authentication logs provide valuable information about:

- Successful user logins
- Failed authentication attempts
- SSH authentication
- Invalid users
- Privilege-related activity
- Authentication failures
- Login/logout activity
- Possible brute-force behaviour

In this lab, we will:

- Understand Linux authentication logs.
- Identify the location and purpose of authentication log files.
- Generate controlled authentication events.
- Analyse successful and failed login events.
- Configure Wazuh to collect Linux authentication logs.
- Verify that logs are reaching Wazuh.
- Investigate authentication events in the Wazuh Dashboard.
- Perform a basic brute-force detection exercise.
- Document findings from a SOC Analyst perspective.

---

# 2. Lab Environment

| Component | Description |
|---|---|
| Linux Endpoint | Ubuntu Linux |
| SIEM | Wazuh |
| Log Collection | Wazuh Agent |
| Authentication Log | `/var/log/auth.log` |
| Log Analysis | Wazuh Dashboard |
| Primary Protocol | SSH |
| Additional Tool | `journalctl` |
| Privileges | sudo/root required for some activities |

### Lab Architecture

```text
                    Linux Endpoint
                  +----------------+
                  |    Ubuntu      |
                  |                |
                  | SSH Login      |
                  | Local Login    |
                  | sudo Activity  |
                  |       |        |
                  |       v        |
                  | /var/log/      |
                  | auth.log       |
                  +-------+--------+
                          |
                          | Wazuh Agent
                          |
                          v
                  +----------------+
                  | Wazuh Manager  |
                  |                |
                  | Log Analysis   |
                  | Decoders       |
                  | Detection      |
                  | Rules          |
                  +-------+--------+
                          |
                          v
                  +----------------+
                  | Wazuh Dashboard|
                  |                |
                  | Authentication |
                  | Investigation  |
                  | Alerts         |
                  +----------------+
```

---

# 3. Understanding Linux Authentication Logs

On Ubuntu and Debian-based Linux systems, authentication-related events are commonly recorded in:

```text
/var/log/auth.log
```

The file may contain information related to:

- SSH authentication
- `sudo` activity
- User authentication
- Failed login attempts
- Successful login attempts
- Invalid users
- PAM authentication events
- Session opening and closing

Example:

```text
Nov 05 10:30:21 ubuntu sshd[1234]: Failed password for invalid user test from 192.168.1.50 port 42110 ssh2
```

This event indicates an unsuccessful SSH authentication attempt.

Another example:

```text
Nov 05 10:35:12 ubuntu sshd[1250]: Accepted password for yash from 192.168.1.50 port 42215 ssh2
```

This indicates a successful SSH authentication.

**Important:** Log messages vary depending on the Linux distribution, SSH configuration, authentication method, and system configuration.

---

# 4. Important Authentication Events

There is no single universal Event ID system equivalent to Windows Security Event IDs.

Linux authentication events are generally identified through:

- Log file
- Service/process
- Message
- Authentication result
- Username
- Source IP
- Timestamp

Common `sshd` messages include:

| Event | Example Indicator | SOC Relevance |
|---|---|---|
| Successful SSH login | `Accepted password` | Identify successful access |
| Successful SSH key login | `Accepted publickey` | Identify key-based access |
| Failed login | `Failed password` | Authentication failure |
| Invalid account | `Invalid user` | Possible scanning/reconnaissance |
| Authentication failure | `authentication failure` | Investigate failed authentication |
| Connection closed | `Connection closed` | Session activity |
| Session opened | `session opened` | User session started |
| Session closed | `session closed` | User session ended |
| sudo activity | `sudo:` | Privilege-related activity |

---

# 5. Step-by-Step Lab Execution

## Step 1: Identify Your Linux Distribution

Open a terminal and run:

```bash
cat /etc/os-release
```

Record:

```text
NAME=
VERSION=
```

For this lab, Ubuntu/Debian-based systems are preferred because `/var/log/auth.log` is commonly available.

---

# 6. Check Authentication Log File

Run:

```bash
ls -lh /var/log/auth.log
```

Then:

```bash
sudo tail -n 20 /var/log/auth.log
```

This displays the latest 20 authentication-related log entries.

You may see entries related to:

- SSH
- sudo
- system services
- user sessions

### Follow the Log in Real Time

Run:

```bash
sudo tail -f /var/log/auth.log
```

This continuously displays new authentication events.

Keep this terminal open.

You can stop the command later using:

```text
Ctrl + C
```

---

# 7. Generate a Successful Local Authentication Event

We first want to understand legitimate authentication activity.

Open another terminal.

Run:

```bash
whoami
```

Then:

```bash
id
```

Then:

```bash
sudo -v
```

Enter your normal Linux password when prompted.

Now return to the terminal running:

```bash
sudo tail -f /var/log/auth.log
```

You should see authentication-related activity associated with the `sudo` command.

Look for fields such as:

- Timestamp
- Username
- Process
- Authentication result
- Command
- Session information

---

# 8. Generate a Controlled Failed Authentication Event

We will intentionally create **one** failed authentication attempt against your own local account.

Do not repeatedly enter incorrect passwords because repeated failures can trigger security controls or account lockout policies.

### Step 1

Open another terminal.

Run:

```bash
sudo -k
```

This clears the cached sudo authentication timestamp.

### Step 2

Run:

```bash
sudo -v
```

When prompted for your password, intentionally enter an incorrect password **once**.

You should receive an authentication failure.

### Step 3

Return to:

```bash
sudo tail -f /var/log/auth.log
```

Look for a new authentication failure.

You may see a message similar to:

```text
sudo: pam_unix(sudo:auth): authentication failure
```

The exact message depends on your Linux distribution and configuration.

---

# 9. Analyse the Authentication Event

For every event you investigate, record:

| Field | Value |
|---|---|
| Timestamp | |
| Hostname | |
| Service | |
| Username | |
| Source IP | |
| Authentication Result | |
| Process/PID | |
| Message | |

For example:

```text
Timestamp:
Service: sudo
Username:
Result: Failed
Source:
Message:
```

### SOC Interpretation

A single failed authentication attempt is normally not sufficient to classify activity as malicious.

A SOC Analyst should look for:

- Repeated failures
- Multiple usernames
- External source IPs
- Successful login after multiple failures
- Unusual login times
- Privilege escalation
- New accounts
- Other related events

---

# 10. Inspect SSH Authentication Logs

SSH is particularly important for SOC monitoring because internet-facing SSH services are frequently targeted by automated login attempts.

First verify whether SSH is installed:

```bash
systemctl status ssh
```

On some distributions, the service may be named:

```bash
sshd
```

You can check using:

```bash
systemctl status sshd
```

If SSH is not installed, do not install it solely for this exercise unless you need an SSH lab environment.

---

# 11. Generate a Successful SSH Login

If SSH is already configured and running, determine your Linux machine's IP address:

```bash
ip addr
```

or:

```bash
hostname -I
```

From another authorized machine on your lab network, connect using SSH:

```bash
ssh <username>@<linux-ip>
```

Example:

```bash
ssh yash@192.168.1.100
```

Use the credentials of the authorized lab account.

After successfully logging in, run:

```bash
whoami
```

Then:

```bash
hostname
```

Then:

```bash
exit
```

Return to the Linux authentication log:

```bash
sudo tail -n 50 /var/log/auth.log
```

Look for messages containing:

```text
Accepted password
```

or:

```text
Accepted publickey
```

depending on your authentication method.

---

# 12. Analyse SSH Authentication Events

A successful SSH event can provide information such as:

```text
User
Source IP
Authentication method
Timestamp
SSH process
Session information
```

Document an event using:

```text
Authentication Type:
Username:
Source IP:
Timestamp:
Result:
Authentication Method:
```

Example:

```text
Authentication Type: SSH
Username: yash
Source IP: 192.168.1.50
Result: Successful
Authentication Method: password
```

Do not publish real usernames or IP addresses unnecessarily in a public GitHub repository.

---

# 13. Search Authentication Logs

Instead of reading the complete log, use `grep` to search for specific authentication events.

### Failed Password Attempts

```bash
sudo grep "Failed password" /var/log/auth.log
```

### Successful Password Logins

```bash
sudo grep "Accepted password" /var/log/auth.log
```

### Public Key Logins

```bash
sudo grep "Accepted publickey" /var/log/auth.log
```

### Invalid Users

```bash
sudo grep "Invalid user" /var/log/auth.log
```

### sudo Activity

```bash
sudo grep "sudo:" /var/log/auth.log
```

### Authentication Failures

```bash
sudo grep "authentication failure" /var/log/auth.log
```

---

# 14. Count Failed SSH Authentication Attempts

Run:

```bash
sudo grep -c "Failed password" /var/log/auth.log
```

This returns the number of matching entries in the current log file.

You can also group failures by source IP:

```bash
sudo grep "Failed password" /var/log/auth.log |
awk '{print $(NF-3)}' |
sort |
uniq -c |
sort -nr
```

The exact field position can vary depending on the log format, so verify the output against actual log entries.

---

# 15. Identify Invalid Users

Run:

```bash
sudo grep "Invalid user" /var/log/auth.log
```

You may see events such as:

```text
Failed password for invalid user admin
```

or:

```text
Invalid user test
```

### SOC Interpretation

Repeated attempts against different usernames may indicate:

- Automated scanning
- Credential stuffing
- Brute-force activity
- Internet-wide SSH scanning

However, one invalid-user event alone does not prove malicious activity.

---

# 16. Inspect Logs Using journalctl

Modern Linux systems may also expose authentication events through `systemd-journald`.

Run:

```bash
sudo journalctl -u ssh
```

If your service is named `sshd`, use:

```bash
sudo journalctl -u sshd
```

To view recent authentication-related journal entries:

```bash
sudo journalctl --since "1 hour ago"
```

You can filter for SSH:

```bash
sudo journalctl --since "1 hour ago" | grep ssh
```

This allows you to compare journal entries with `/var/log/auth.log`.

---

# 17. Configure Wazuh Agent for Authentication Logs

Now we will configure the Wazuh Agent to collect Linux authentication logs.

## Step 1: Verify Wazuh Agent

On the Linux endpoint, check the agent service:

```bash
sudo systemctl status wazuh-agent
```

The expected state is:

```text
Active: active (running)
```

If the agent is not running, start it using:

```bash
sudo systemctl start wazuh-agent
```

Enable it at boot:

```bash
sudo systemctl enable wazuh-agent
```

---

# 18. Configure Wazuh Agent

The Wazuh Agent configuration file is normally located at:

```text
/var/ossec/etc/ossec.conf
```

Before editing the configuration, create a backup:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup
```

Open the configuration:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Locate the existing:

```xml
<ossec_config>
```

section.

Add the following configuration inside the existing configuration:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>
```

Do not create another outer `<ossec_config>` element.

Do not remove existing Wazuh configuration.

Save the file.

---

# 19. Restart Wazuh Agent

Restart the agent:

```bash
sudo systemctl restart wazuh-agent
```

Verify:

```bash
sudo systemctl status wazuh-agent
```

The service should be running.

---

# 20. Verify Wazuh Agent Configuration

Check the Wazuh Agent log:

```bash
sudo tail -n 50 /var/ossec/logs/ossec.log
```

Look for configuration or connection errors.

You can also check the Wazuh Agent process:

```bash
sudo systemctl is-active wazuh-agent
```

Expected output:

```text
active
```

---

# 21. Generate a New Authentication Event

Now that Wazuh is configured, generate another harmless authentication event.

Run:

```bash
sudo -k
```

Then:

```bash
sudo -v
```

Enter your correct password.

After that, generate one failed authentication event using:

```bash
sudo -k
sudo -v
```

Enter an incorrect password once.

Then stop.

Do not repeatedly generate failures.

---

# 22. Verify Authentication Events in Wazuh

Open the Wazuh Dashboard.

Navigate to:

```text
Threat Hunting
```

or the equivalent event exploration interface available in your Wazuh version.

Set the time range to include the time when you generated the authentication events.

Search for the Linux endpoint.

Look for authentication-related events from:

```text
/var/log/auth.log
```

Depending on your Wazuh version and decoder output, useful fields may include:

```text
agent.name
agent.id
location
decoder.name
data.srcip
data.dstuser
data.srcuser
data.status
data.system_name
```

The exact field names can vary according to the Wazuh decoder and event type.

---

# 23. Investigate a Failed Authentication Event in Wazuh

Select one failed authentication event.

Open its expanded event/JSON details.

Identify:

```text
Agent:
Timestamp:
Log Location:
Decoder:
Event Message:
Username:
Source IP:
Authentication Result:
```

Document the event:

```text
Agent:
Timestamp:
Source IP:
Username:
Authentication Result:
Authentication Method:
Log Source:
Event Message:
```

---

# 24. Understand Wazuh Alerts

An important SOC concept is the difference between:

```text
Log Event
```

and:

```text
Wazuh Alert
```

A Linux authentication event may be successfully collected by the Wazuh Agent without necessarily generating an alert.

For example:

```text
Failed SSH login
        |
        v
auth.log
        |
        v
Wazuh Agent
        |
        v
Wazuh Manager
        |
        +----> Event collected
        |
        +----> Detection rule matched?
                    |
              +-----+-----+
              |           |
             Yes          No
              |           |
              v           v
           Alert       Event only
```

Therefore, if you can see the event in collected data but do not see a Wazuh alert, this does not necessarily indicate a collection failure.

---

# 25. Brute-Force Investigation Exercise

## Scenario

A Linux server is receiving repeated failed SSH authentication attempts.

As a SOC Analyst, your task is to determine whether the activity resembles an SSH brute-force attack.

### Investigation Steps

1. Open Wazuh Dashboard.
2. Select the Linux endpoint.
3. Search for authentication-related events.
4. Identify failed SSH authentication events.
5. Record the source IP addresses.
6. Record the usernames being targeted.
7. Check the timestamps.
8. Determine whether failures are repeated.
9. Check whether a successful login occurred after multiple failures.
10. Correlate the activity with other Linux authentication events.

### Investigation Questions

Answer the following:

```text
1. How many failed authentication attempts were observed?

2. Were the attempts associated with one username or multiple usernames?

3. Was the source IP consistent?

4. Did the source IP eventually achieve successful authentication?

5. Was the activity expected?

6. Does the pattern resemble brute-force behaviour?

7. What additional evidence would you collect?
```

---

# 26. Controlled Brute-Force Simulation

For this project, do **not** perform a real brute-force attack against external systems.

If you want to demonstrate repeated failed authentication, use only your own isolated Linux lab environment.

Generate a small number of controlled failed authentication events manually, rather than using automated password-guessing tools.

For example:

```text
Attempt 1 → Incorrect password
Attempt 2 → Incorrect password
Attempt 3 → Incorrect password
```

Then stop.

Check:

```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 10
```

Your objective is to understand the resulting logs, not to perform password cracking.

---

# 27. Analyse Source IP Addresses

For SSH-related authentication events:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Look for entries containing:

```text
from <IP_ADDRESS>
```

For example:

```text
Failed password for user1 from 192.168.1.50
```

Record:

```text
Username:
Source IP:
Timestamp:
Result:
```

Then compare the source IP against successful authentication events:

```bash
sudo grep "Accepted" /var/log/auth.log
```

This can help answer:

> Did an IP address that generated multiple failures later successfully authenticate?

This type of correlation is highly relevant to SOC investigations.

---

# 28. Wazuh Investigation Workflow

Use the following workflow when investigating Linux authentication events:

```text
             Authentication Event
                     |
                     v
              Identify Host
                     |
                     v
              Identify User
                     |
                     v
             Identify Source IP
                     |
                     v
             Check Result
          +----------+----------+
          |                     |
        Failed               Successful
          |                     |
          v                     v
   Count Attempts        Verify Legitimacy
          |
          v
   Check Time Pattern
          |
          v
   Correlate Other Events
          |
          v
      Determine Risk
          |
          v
     Document Findings
```

---

# 29. SOC Investigation Report Template

Create a report using the following format:

```text
Incident Title:
Linux Authentication Investigation

Date:
Analyst:

Affected Host:
Wazuh Agent ID:

Log Source:
/var/log/auth.log

Event Type:
SSH / sudo / PAM / Other

Username:

Source IP:

First Observed:
Last Observed:

Authentication Attempts:
Successful:
Failed:

Timeline:

[Time] - Event
[Time] - Event
[Time] - Event

Observations:

1.
2.
3.

Analysis:

Was the activity expected?

Was there evidence of repeated authentication failures?

Was a successful login observed after failures?

Was the source IP authorized?

Risk Assessment:

Low / Medium / High / Undetermined

Conclusion:

Recommended Actions:

1.
2.
3.
```

---

# 30. Screenshots to Capture for GitHub

Create:

```text
02-Log-Collection/Linux-Logs/screenshots/
```

Recommended screenshots:

| Screenshot | Description |
|---|---|
| `01-auth-log.png` | `/var/log/auth.log` output |
| `02-failed-authentication.png` | Controlled failed authentication event |
| `03-successful-ssh-login.png` | Successful SSH authentication |
| `04-ssh-log-analysis.png` | `grep` analysis of SSH events |
| `05-journalctl-authentication.png` | Authentication events using journalctl |
| `06-wazuh-agent-config.png` | Wazuh `auth.log` collection configuration |
| `07-wazuh-agent-status.png` | Active Wazuh Agent |
| `08-wazuh-auth-event.png` | Authentication event visible in Wazuh |
| `09-wazuh-event-details.png` | Expanded Wazuh event details |
| `10-wazuh-investigation.png` | Authentication investigation in Wazuh |

### Screenshot Guidelines

Before uploading screenshots to GitHub:

- Hide personal usernames if necessary.
- Avoid exposing passwords or secrets.
- Review IP addresses.
- Remove API keys or tokens.
- Avoid publishing unrelated system information.
- Keep screenshots focused on the security investigation.

---

# 31. Troubleshooting

## Problem 1: `/var/log/auth.log` Does Not Exist

Check your Linux distribution:

```bash
cat /etc/os-release
```

Check whether journald contains SSH authentication events:

```bash
sudo journalctl -u ssh
```

or:

```bash
sudo journalctl -u sshd
```

Some Linux distributions rely primarily on `journald` rather than `/var/log/auth.log`.

---

## Problem 2: Wazuh Agent Is Not Running

Check:

```bash
sudo systemctl status wazuh-agent
```

Restart:

```bash
sudo systemctl restart wazuh-agent
```

Check:

```bash
sudo systemctl is-active wazuh-agent
```

---

## Problem 3: Authentication Logs Are Not Appearing in Wazuh

Check the agent configuration:

```bash
sudo grep -n -A4 -B2 "/var/log/auth.log" /var/ossec/etc/ossec.conf
```

Verify that the configuration contains:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>
```

Then restart:

```bash
sudo systemctl restart wazuh-agent
```

Check the agent log:

```bash
sudo tail -n 50 /var/ossec/logs/ossec.log
```

---

# 32. Security Considerations

Authentication logs can contain sensitive information such as:

- Usernames
- Internal IP addresses
- Hostnames
- Authentication metadata
- Privilege-related information

Do not publish raw authentication logs without reviewing them.

For the GitHub project, use sanitized screenshots and example values where necessary.

---

# 33. Key Takeaways

After completing this lab, you should understand:

- The purpose of Linux authentication logs.
- The role of `/var/log/auth.log`.
- How SSH authentication appears in Linux logs.
- How successful and failed authentication events differ.
- How to identify invalid users.
- How to analyse authentication events using `grep`.
- How to use `journalctl` for authentication investigation.
- How to configure Wazuh Agent to collect authentication logs.
- How authentication events reach Wazuh.
- The difference between collected events and Wazuh alerts.
- How to investigate potential SSH brute-force behaviour.
- How to document authentication incidents from a SOC Analyst perspective.

---

# 34. References

## Linux

- Linux `journalctl` documentation:
  https://man7.org/linux/man-pages/man1/journalctl.1.html

- Linux `sshd` documentation:
  https://man7.org/linux/man-pages/man8/sshd.8.html

## Wazuh

- Wazuh Log Data Collection:
  https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/index.html

- Wazuh Agent Configuration:
  https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html

- Wazuh Linux Log Collection:
  https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html

---

**Project:** SOC Sentinel – SIEM-Based Log Monitoring & Security Incident Investigation Lab

**Section:** 02 – Log Collection

**Category:** Linux Logs

**Log Source:** `/var/log/auth.log`

**SIEM:** Wazuh

**Status:** In Progress – Update after completing the practical lab.
