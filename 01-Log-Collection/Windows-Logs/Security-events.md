# Windows Security Events

## 1. Overview

Windows Security Event Logs record security-related activities on a Windows system, including authentication attempts, account management, privilege use, and security policy changes.

In a Security Operations Center (SOC), analysts use these logs to investigate suspicious login activity, identify potential brute-force attempts, monitor account changes, and correlate events during security investigations.

**Objective:** Understand important Windows Security Event IDs, locate relevant logs, and prepare them for analysis in a SIEM such as Splunk.

## 2. Log Location

Windows Security events can be viewed using Event Viewer.

**Navigation:**
1. Press Win + R.
2. Type `eventvwr.msc`.
3. Navigate to **Windows Logs → Security**.
4. Select an event to inspect its details.

The underlying log file is typically located at:

`C:\Windows\System32\winevt\Logs\Security.evtx`

Access to the Security log may require administrator privileges.

## 3. Important Windows Security Event IDs

| Event ID | Description | SOC Relevance |
|---|---|---|
| 4624 | Successful logon | Identify successful authentication |
| 4625 | Failed logon | Investigate failed login attempts |
| 4634 | Account logged off | Understand session activity |
| 4647 | User initiated logoff | Review user session termination |
| 4648 | Logon using explicit credentials | Investigate use of supplied credentials |
| 4672 | Special privileges assigned to a new logon | Review privileged sessions |
| 4688 | New process created | Investigate process execution; command-line details require appropriate auditing |
| 4697 | Service installed on a system | Investigate unexpected service installation |
| 4720 | User account created | Monitor account creation |
| 4722 | User account enabled | Investigate account reactivation |
| 4724 | Attempt to reset an account password | Review password reset activity |
| 4728 | Member added to a security-enabled global group | Monitor group membership changes |
| 4732 | Member added to a security-enabled local group | Monitor local group changes |
| 4738 | User account changed | Investigate account modifications |
| 4740 | User account locked out | Investigate repeated authentication failures |
| 1102 | Security audit log cleared | Investigate possible log tampering |

**Note:** Event availability depends on Windows version, activity performed, audit policy, and system configuration. Event IDs alone do not prove malicious activity.

## 4. Key Fields to Examine

When opening a Windows Security event, inspect the following fields where available:

- **TimeCreated:** When the event occurred.
- **Computer:** Host that recorded the event.
- **Event ID:** Type of security event.
- **Account Name:** User associated with the event.
- **Logon Type:** Type of logon, particularly for authentication events.
- **Source Network Address:** Source IP, if recorded.
- **Workstation Name:** Originating workstation, if recorded.
- **Process Name:** Associated process, where applicable.
- **Status / SubStatus:** Authentication failure details, where applicable.

Field availability and meaning vary by event type.

## 5. Practical Lab: Investigating Failed Logons

### Objective

Generate a small number of controlled failed sign-in attempts on your own Windows lab machine and examine the resulting Security events.

### Procedure

1. Open Event Viewer.
2. Navigate to **Windows Logs → Security**.
3. Filter the current log for Event ID `4625`.
4. If no matching events appear, perform a controlled failed sign-in using a test account or another authorized method.
5. Refresh the log and inspect the newly generated event.
6. Record the timestamp, target account, logon type, status/substatus, and source address if available.
7. Take a screenshot with sensitive information removed.

Do not repeatedly enter incorrect passwords on your everyday account; account lockout policies may apply.

### Observation Template

| Field | Observation |
|---|---|
| Event ID | 4625 |
| Timestamp | Record from your event |
| Target Account | Record from your event |
| Logon Type | Record from your event |
| Source Address | Record if available |
| Status / SubStatus | Record if available |
| Initial Assessment | Failed authentication; investigate context |

## 6. Authentication Event Investigation

A SOC analyst should correlate failed and successful logons instead of judging an event in isolation.

For example:

1. Identify repeated Event ID 4625 events.
2. Compare their timestamps and account names.
3. Check whether the same source address appears repeatedly.
4. Look for a subsequent Event ID 4624 involving the same account or source, where available.
5. Examine logon type and surrounding activity.
6. Decide whether the activity appears expected, suspicious, or inconclusive.

A successful login after repeated failures warrants further investigation, but it does not by itself establish account compromise.

## 7. SIEM Integration

When Windows Security logs are forwarded to Splunk, the following search can be used as a starting point. Adjust the index and field names to match your actual configuration.

```spl
index=windows EventCode=4625
| table _time host user src EventCode
| sort - _time
```

This search assumes the events are indexed under `index=windows` and that the relevant fields have been extracted. Your environment may use `EventID` instead of `EventCode`, or different field names.

### Count Failed Logons by User

```spl
index=windows EventCode=4625
| stats count by user
| sort - count
```

### Review Successful Logons

```spl
index=windows EventCode=4624
| table _time host user src EventCode
| sort - _time
```

These are starting searches, not complete brute-force detections. A production detection should account for time windows, source addresses, target accounts, event volume, and false positives.

## 8. SOC Investigation Checklist

- [ ] Identify the event ID and timestamp.
- [ ] Determine the affected host and account.
- [ ] Review the logon type and authentication result.
- [ ] Examine source address and workstation details when available.
- [ ] Correlate related failed and successful logons.
- [ ] Check for account lockouts or privilege-related events.
- [ ] Assess whether the activity is expected or suspicious.
- [ ] Document evidence and recommended next steps.

## 9. Evidence and Screenshots
<img width="1302" height="833" alt="image" src="https://github.com/user-attachments/assets/824ed9eb-7427-45a5-9b95-75907af0113a" />


Only add screenshots after capturing the corresponding events in your own lab.

## 10. Key Takeaways

- Windows Security logs provide evidence of authentication and account-related activity.
- Event ID 4625 records failed logons; Event ID 4624 records successful logons.
- Event IDs must be interpreted alongside timestamps, account details, logon types, and surrounding events.
- Splunk can help search, aggregate, and correlate Windows Security events.
- SOC analysts should document evidence, assess context, and escalate suspicious activity according to the incident-response procedure.

## 11. References

- Microsoft Learn — [Audit Logon](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-logon)
- Microsoft Learn — [Advanced Security Audit Policy Settings](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings)
- Splunk Documentation — [Search Manual](https://docs.splunk.com/Documentation/Splunk/latest/Search/Aboutthesearchlanguage)

---

**Project:** SOC Sentinel — SIEM-Based Log Monitoring & Security Incident Investigation Lab

**Focus:** Windows Event Logging | SIEM | Splunk | Authentication Monitoring | SOC L1 Investigation
