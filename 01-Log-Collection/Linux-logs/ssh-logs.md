# Linux SSH Logs – Collection and Analysis Using Wazuh

## 1. Objective

The objective of this lab is to understand SSH logging on Linux systems, collect SSH-related events using the Wazuh Agent, and investigate successful and failed SSH authentication activity through the Wazuh Dashboard.

SSH is commonly used for remote Linux administration. Because exposed SSH services can be targeted by attackers, monitoring SSH authentication is an important SOC activity.

In this lab, we will:

- Understand SSH logging.
- Verify the SSH service.
- Identify SSH authentication events.
- Generate controlled successful and failed SSH logins.
- Analyse SSH logs using command-line tools.
- Configure Wazuh to collect SSH-related logs.
- Verify SSH events in Wazuh.
- Investigate repeated authentication failures.
- Correlate failed and successful SSH activity.
- Document the investigation as a SOC Analyst.

---

# 2. Lab Environment

| Component | Description |
|---|---|
| Linux Endpoint | Ubuntu Linux |
| SIEM | Wazuh |
| Log Collection | Wazuh Agent |
| SSH Service | OpenSSH |
| Authentication Log | `/var/log/auth.log` |
| Log Analysis | Wazuh Dashboard |
| Additional Tool | `journalctl` |
| Privileges | sudo/root required for some activities |

---

# 3. Lab Architecture

```text
                 SSH Client
                     |
                     | SSH
                     v
+--------------------------------+
|          Linux Server          |
|                                |
|       OpenSSH Server           |
|              |                 |
|              v                 |
|       Authentication Logs      |
|              |                 |
|              v                 |
|       /var/log/auth.log        |
|              |                 |
|              v                 |
|         Wazuh Agent            |
+--------------|-----------------+
               |
               | Log Collection
               v
+--------------------------------+
|         Wazuh Manager          |
|                                |
| Decoders / Detection Rules     |
| Alert Processing               |
+--------------|-----------------+
               |
               v
+--------------------------------+
|        Wazuh Dashboard         |
|                                |
| SSH Monitoring                 |
| Authentication Investigation   |
| Brute-Force Detection          |
+--------------------------------+
```

---

# 4. Understanding SSH

SSH (Secure Shell) provides encrypted remote access to Linux systems.

The SSH server is commonly provided by:

```text
OpenSSH
```

The SSH server process is generally:

```text
sshd
```

SSH authentication events can contain information such as:

- Username
- Source IP address
- Authentication method
- Authentication result
- Timestamp
- SSH process
- Session status

---

# 5. Check Whether SSH Is Installed

Run:

```bash
ssh -V
```

Then check the SSH server:

```bash
systemctl status ssh
```

If that service does not exist, check:

```bash
systemctl status sshd
```

If neither service exists, your machine may not be configured as an SSH server.

---

# 6. Identify the SSH Listening Port

Run:

```bash
sudo ss -tlnp | grep ssh
```

You may see something similar to:

```text
LISTEN 0 128 0.0.0.0:22
```

Port `22` is the standard SSH port, although administrators can configure SSH to use another port.

Record:

```text
SSH Service:
SSH Port:
Listening Address:
```

---

# 7. Check SSH Configuration

The primary SSH server configuration file is:

```text
/etc/ssh/sshd_config
```

You can inspect relevant settings using:

```bash
sudo grep -E "^(Port|PermitRootLogin|PasswordAuthentication|PubkeyAuthentication)" /etc/ssh/sshd_config
```

Do not modify SSH security settings during this lab unless you specifically intend to test a configuration change.

---

# 8. Monitor SSH Authentication Logs

On Ubuntu/Debian systems, SSH authentication events are commonly recorded in:

```text
/var/log/auth.log
```

Run:

```bash
sudo grep "sshd" /var/log/auth.log | tail -n 20
```

This displays recent SSH-related events.

You can monitor them in real time:

```bash
sudo tail -f /var/log/auth.log
```

Keep this terminal open while performing the exercises.

---

# 9. Successful SSH Authentication

Use another authorized machine on your lab network as the SSH client.

First identify the Linux server's IP:

```bash
hostname -I
```

or:

```bash
ip addr
```

From the authorized client:

```bash
ssh <username>@<server-ip>
```

Example:

```bash
ssh yash@192.168.1.100
```

Enter the legitimate password or use your configured SSH key.

After connecting:

```bash
whoami
```

Then:

```bash
hostname
```

Then exit:

```bash
exit
```

---

# 10. Identify the Successful SSH Event

On the Linux server run:

```bash
sudo grep "Accepted" /var/log/auth.log | tail -n 20
```

You may find an entry similar to:

```text
sshd: Accepted password for user1 from 192.168.1.50
```

or:

```text
sshd: Accepted publickey for user1 from 192.168.1.50
```

Record:

```text
Timestamp:
Username:
Source IP:
Authentication Method:
Result:
```

---

# 11. Controlled Failed SSH Authentication

We will now generate a single controlled failed SSH authentication event.

Use an authorized SSH client connected to your own lab server.

Attempt:

```bash
ssh <username>@<server-ip>
```

When prompted for the password, enter an incorrect password **once**.

Then stop.

Do not repeatedly guess passwords against systems.

Return to the Linux server and run:

```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 20
```

You should see a corresponding failed authentication event.

---

# 12. Analyse the Failed SSH Event

A typical event may contain information similar to:

```text
Failed password for user1 from 192.168.1.50
```

Record:

```text
Timestamp:
Username:
Source IP:
Authentication Method:
Result:
```

### SOC Interpretation

One failed login attempt is not sufficient evidence of a brute-force attack.

A SOC Analyst should examine:

- Number of failures
- Time interval between attempts
- Source IP
- Target usernames
- Whether multiple accounts were targeted
- Whether a successful login followed
- Whether the source IP is expected
- Whether other suspicious activity occurred

---

# 13. Investigate Invalid SSH Users

Search for invalid-user attempts:

```bash
sudo grep "Invalid user" /var/log/auth.log | tail -n 20
```

You may see activity such as:

```text
Invalid user admin
```

or:

```text
Invalid user test
```

Repeated attempts against different usernames may indicate automated scanning or brute-force behaviour.

However, the activity should be correlated with additional evidence before classifying it as malicious.

---

# 14. Analyse SSH Authentication Methods

Search for password authentication:

```bash
sudo grep "Accepted password" /var/log/auth.log
```

Search for public-key authentication:

```bash
sudo grep "Accepted publickey" /var/log/auth.log
```

Search for failed password attempts:

```bash
sudo grep "Failed password" /var/log/auth.log
```

This allows you to distinguish successful and failed authentication methods.

---

# 15. Analyse SSH Activity Using journalctl

If your Linux system uses systemd, you can query SSH logs through `journalctl`.

Run:

```bash
sudo journalctl -u ssh
```

If your service is named `sshd`:

```bash
sudo journalctl -u sshd
```

To view recent SSH activity:

```bash
sudo journalctl -u ssh --since "1 hour ago"
```

You can also search for failed authentication:

```bash
sudo journalctl -u ssh | grep "Failed"
```

And successful authentication:

```bash
sudo journalctl -u ssh | grep "Accepted"
```

---

# 16. Count Failed SSH Attempts

Run:

```bash
sudo grep -c "Failed password" /var/log/auth.log
```

This gives the number of matching entries in the current authentication log.

You can also inspect recent failures:

```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 20
```

---

# 17. Identify Source IPs

Search failed SSH attempts:

```bash
sudo grep "Failed password" /var/log/auth.log
```

Look for:

```text
from <IP_ADDRESS>
```

For example:

```text
Failed password for user1 from 192.168.1.50
```

Record the source IP.

Then search successful authentication:

```bash
sudo grep "Accepted" /var/log/auth.log
```

This allows you to determine whether a source associated with failed authentication later achieved successful access.

---

# 18. Basic SSH Brute-Force Investigation

## Scenario

Your Linux server is receiving multiple failed SSH authentication attempts.

A SOC Analyst must determine whether this activity represents normal user error or possible brute-force behaviour.

### Investigation Steps

1. Identify the Linux endpoint.
2. Identify the SSH service.
3. Search for failed authentication.
4. Count the failures.
5. Identify source IP addresses.
6. Identify targeted usernames.
7. Check timestamps.
8. Determine whether the attempts are concentrated in a short period.
9. Search for successful authentication from the same source.
10. Correlate the activity with other Linux logs.
11. Document the findings.

---

# 19. Controlled Brute-Force Pattern Exercise

For this lab, simulate a suspicious pattern **without using password-cracking software or attacking external systems**.

Use your own isolated Linux environment.

Generate a small number of controlled failed SSH attempts:

```text
Attempt 1 → Incorrect password
Attempt 2 → Incorrect password
Attempt 3 → Incorrect password
```

Then stop.

Check the resulting events:

```bash
sudo grep "Failed password" /var/log/auth.log | tail -n 10
```

You should now have multiple authentication failures that can be investigated.

The objective is to understand how repeated authentication failures appear in logs.

---

# 20. Configure Wazuh Agent

Now configure Wazuh to collect the Linux authentication log containing SSH events.

Because SSH authentication is commonly recorded in:

```text
/var/log/auth.log
```

the Wazuh configuration can use the same log collection mechanism used in the authentication-log lab.

## Step 1: Check Wazuh Agent

```bash
sudo systemctl status wazuh-agent
```

If necessary:

```bash
sudo systemctl start wazuh-agent
```

---

# 21. Back Up Wazuh Configuration

Run:

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.backup
```

---

# 22. Configure SSH Log Collection

Open:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Inside the existing `<ossec_config>` section, ensure the authentication log collection exists:

```xml
<localfile>
  <location>/var/log/auth.log</location>
  <log_format>syslog</log_format>
</localfile>
```

Do not create another `<ossec_config>` section.

If you already configured this in `auth-logs.md`, **do not add a duplicate entry**.

The same Wazuh Agent configuration can collect both:

```text
Linux Authentication Events
        +
SSH Authentication Events
        |
        v
/var/log/auth.log
        |
        v
Wazuh Agent
```

---

# 23. Restart Wazuh Agent

Run:

```bash
sudo systemctl restart wazuh-agent
```

Check:

```bash
sudo systemctl status wazuh-agent
```

Verify:

```bash
sudo systemctl is-active wazuh-agent
```

Expected:

```text
active
```

---

# 24. Generate a New SSH Event

Perform a legitimate SSH login from your authorized lab client.

Then generate one controlled failed login.

Return to the server and confirm locally:

```bash
sudo grep "sshd" /var/log/auth.log | tail -n 20
```

Now the events should be available for Wazuh collection.

---

# 25. Verify SSH Events in Wazuh

Open the Wazuh Dashboard.

Navigate to:

```text
Threat Hunting
```

or the equivalent event exploration interface.

Set the time range around your SSH test.

Select the Linux endpoint.

Search for SSH-related events.

Useful information may include:

```text
agent.name
agent.id
location
decoder.name
data.srcip
data.dstuser
data.srcuser
data.status
```

The exact fields depend on the Wazuh decoder and version.

Open an event and inspect its expanded JSON/details.

---

# 26. Investigate Failed SSH Authentication in Wazuh

Select a failed SSH authentication event.

Record:

```text
Agent:
Timestamp:
Source IP:
Username:
Authentication Result:
Log Location:
Decoder:
Event Message:
```

Then investigate:

```text
How many failures came from this IP?
Was the same username targeted?
Were multiple usernames targeted?
Did successful authentication occur afterward?
Was the source IP expected?
```

---

# 27. Identify a Potential Brute-Force Pattern in Wazuh

Use Wazuh to search for repeated SSH failures.

The exact search syntax depends on the Wazuh Dashboard version and fields available in your installation.

Start by filtering the Linux agent and authentication-related events.

Look for repeated events containing:

```text
Failed password
```

or:

```text
Invalid user
```

Then compare:

```text
Source IP
Username
Timestamp
Frequency
Authentication result
```

### Example Investigation Logic

```text
Multiple Failed SSH Attempts
             |
             v
      Same Source IP?
        /          \
      Yes           No
       |             |
       v             v
   Check User    Distributed
   & Frequency   Activity
       |
       v
Successful Login?
     /     \
   Yes      No
    |        |
    v        v
Priority   Continue
Investigation
```

---

# 28. Correlate SSH With Authentication Logs

SSH events are part of the broader Linux authentication picture.

Compare:

```bash
sudo grep "sshd" /var/log/auth.log
```

with:

```bash
sudo grep "sudo:" /var/log/auth.log
```

and:

```bash
sudo grep "authentication failure" /var/log/auth.log
```

The goal is to determine whether SSH activity was followed by:

- sudo activity
- privilege-related activity
- additional authentication attempts
- session creation
- unexpected system changes

This is an important SOC investigation technique.

---

# 29. Correlate SSH With Syslog

If suspicious SSH activity occurred around a particular time, inspect general system logs.

For example:

```bash
sudo grep "Nov 13 10:35" /var/log/syslog
```

And:

```bash
sudo grep "Nov 13 10:35" /var/log/auth.log
```

Replace the date/time with the timestamp from your actual investigation.

This allows you to correlate:

```text
SSH Authentication
        |
        v
Authentication Result
        |
        v
System Activity
        |
        v
Potential Security Event
```

---

# 30. SOC Investigation Report

Use the following template:

```text
Incident Title:
SSH Authentication Investigation

Date:
Analyst:

Affected Host:
Wazuh Agent ID:

SSH Service:
SSH Port:

First Observed:
Last Observed:

Source IP:

Target Username(s):

Successful Attempts:

Failed Attempts:

Invalid User Attempts:

Authentication Methods:

Timeline:

[Time] - Failed authentication
[Time] - Failed authentication
[Time] - Successful authentication
[Time] - Session activity

Observations:

1.
2.
3.

Correlation:

Authentication Logs:
Syslog:
Other Events:

Analysis:

Does the activity resemble brute-force behaviour?

Was successful access achieved?

Is the source IP authorized?

Risk Assessment:

Low / Medium / High / Undetermined

Conclusion:

Recommended Actions:

1.
2.
3.
```

---

# 31. Screenshots for GitHub

Create:

```text
02-Log-Collection/Linux-Logs/screenshots/
```

Recommended screenshots:

| Screenshot | Description |
|---|---|
| `01-ssh-service-status.png` | SSH service status |
| `02-ssh-listening-port.png` | SSH listening socket |
| `03-successful-ssh-login.png` | Successful SSH login |
| `04-failed-ssh-login.png` | Controlled failed SSH login |
| `05-ssh-auth-log.png` | SSH events in `auth.log` |
| `06-ssh-grep-analysis.png` | SSH log analysis using grep |
| `07-journalctl-ssh.png` | SSH events using journalctl |
| `08-wazuh-agent-config.png` | Wazuh authentication log configuration |
| `09-wazuh-ssh-event.png` | SSH event visible in Wazuh |
| `10-wazuh-ssh-details.png` | Expanded Wazuh event |
| `11-wazuh-bruteforce-investigation.png` | Investigation of repeated failures |

Review screenshots before publishing them.

Do not expose:

- Passwords
- SSH private keys
- API keys
- Tokens
- Sensitive internal information
- Unnecessary personal information

---

# 32. Security Considerations

SSH is a security-sensitive service.

During this lab:

- Use only systems you own or are explicitly authorized to test.
- Do not perform brute-force attacks against external systems.
- Do not use automated password-cracking tools for this exercise.
- Do not expose your SSH service to the Internet merely to generate logs.
- Do not weaken SSH security settings for the lab.
- Do not publish SSH private keys.
- Do not publish passwords.

The goal is **log collection and SOC investigation**, not password cracking.

---

# 33. Key Takeaways

After completing this lab, you should understand:

- How SSH works at a high level.
- Where SSH authentication events are recorded.
- How successful SSH authentication appears in Linux logs.
- How failed SSH authentication appears in Linux logs.
- How to identify invalid SSH users.
- How to identify source IP addresses.
- How to correlate failed and successful authentication.
- How to investigate possible brute-force patterns.
- How Wazuh collects SSH-related authentication events.
- How to investigate SSH activity using Wazuh.
- How to correlate SSH events with authentication and syslog activity.
- How to document an SSH security investigation as a SOC Analyst.

---

# 34. References

### OpenSSH / Linux

- OpenSSH Server documentation:
  https://man.openbsd.org/sshd

- `sshd_config` documentation:
  https://man.openbsd.org/sshd_config

- Linux `journalctl` documentation:
  https://man7.org/linux/man-pages/man1/journalctl.1.html

### Wazuh

- Wazuh Log Data Collection:
  https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/index.html

- Wazuh Agent `localfile` Configuration:
  https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html

---
**Log Source:** `/var/log/auth.log` / SSH

**SIEM:** Wazuh

**Status:** In Progress – Update after completing the practical lab.
