# Windows System Events — Monitoring and Investigation

## 1. Overview

Windows System Event Logs record activities involving operating system components, services, drivers, updates, and system startup or shutdown.

For a SOC Analyst L1, these events help investigate unexpected restarts, service installation, system failures, and unusual changes to the host.

**Objective:** Learn to locate Windows System logs, understand important Event IDs, investigate system activity, and prepare events for SIEM analysis using Splunk.

## 2. Log Location

1. Press Win + R.
2. Enter `eventvwr.msc`.
3. Navigate to **Event Viewer → Windows Logs → System**.
4. Review the events and select one to view its General and Details tabs.

The underlying log file is typically:

`C:\Windows\System32\winevt\Logs\System.evtx`

## 3. Important Windows System Event IDs

| Event ID | Provider / Source | Description | SOC Relevance |
|---|---|---|---|
| 12 | Kernel-General | Operating system started | Review startup history |
| 13 | Kernel-General | Operating system shutting down | Review shutdown activity |
| 19 | WindowsUpdateClient | Successful update installation in commonly documented scenarios | Correlate updates with system changes |
| 41 | Kernel-Power | System restarted without a clean shutdown | Investigate unexpected restart |
| 1001 | WER-SystemErrorReporting | Bugcheck report in relevant scenarios | Investigate system crash |
| 1074 | User32 | Process or user initiated shutdown/restart | Identify the initiating process and reason |
| 6005 | EventLog | Event Log service started | Review startup timeline |
| 6006 | EventLog | Event Log service stopped | Review orderly shutdown activity |
| 6008 | EventLog | Previous shutdown was unexpected | Investigate unexpected shutdown |
| 6009 | EventLog | Operating system version information logged at startup | Correlate startup events |
| 7036 | Service Control Manager | A service entered a particular state | Review service state changes |
| 7045 | Service Control Manager | A service was installed | Investigate unexpected service installation |

**Important:** Interpret an Event ID together with its provider, message, timestamp, and surrounding events. Event IDs are not globally unique across all providers, and individual events do not prove malicious activity.

## 4. Important Fields to Examine

When inspecting an event, record:

- **TimeCreated:** When the event was recorded.
- **Provider / Source:** Component that generated the event.
- **Event ID:** Identifier for the event type.
- **Level:** Information, Warning, Error, or Critical.
- **Computer:** Host that recorded the event.
- **User:** Account associated with the event, if available.
- **General / Message:** Human-readable description.
- **Event Data:** Additional fields specific to the event.

## 5. Practical Lab A — Explore System Logs

### Objective

Explore the System log and identify recent startup, shutdown, service, and update events.

### Steps

1. Open Event Viewer.
2. Navigate to Windows Logs → System.
3. Sort by Date and Time if necessary.
4. Select a recent event.
5. Read its provider, Event ID, level, and message.
6. Find events with IDs 12, 13, 6005, or 6006 if available.
7. Record the events and take a screenshot.

### Observation Table

| Timestamp | Provider | Event ID | Level | Summary |
|---|---|---|---|---|
| Record from your lab | Record from event | Record from event | Record from event | Summarize the message |

Do not invent events if a particular ID is absent from your system.

## 6. Practical Lab B — Investigate a Normal Restart

### Objective

Understand how Windows records a planned restart.

### Steps

1. Save your work and close important applications.
2. Restart your own computer normally using the Windows Start menu.
3. Sign back in after the restart.
4. Open Event Viewer → Windows Logs → System.
5. Filter for Event IDs `1074,6005,6006,6008`.
6. Inspect the timestamps and event messages.
7. Look for a planned restart event and the subsequent startup or Event Log service events, if present.

### What to investigate

- Which process initiated the restart?
- Which account was associated with it?
- What reason was recorded?
- Were the events consistent with a planned restart?

Not every system produces every event in the same sequence. Record what your computer actually shows.

## 7. Practical Lab C — Investigate Service Events

### Objective

Understand service state changes and service installation records.

### Steps

1. Return to Windows Logs → System.
2. Filter for Event IDs `7036,7045`.
3. Open a matching event if one exists.
4. For Event ID 7036, review the service name and reported state.
5. For Event ID 7045, review the service name, executable path, start type, and service account where available.
6. Record whether the service or change is familiar and expected.

### SOC investigation questions

- Is the service associated with known software?
- Does its executable path match the expected installation location?
- Was the service installation expected?
- Does the timestamp correspond with a software or driver update?
- Is additional verification needed?

A legitimate application or update can install a service. An unfamiliar name alone is not sufficient evidence of malware.

**Safety note:** Inspect existing events only. Do not disable or remove services as part of this beginner lab.

## 8. Practical Lab D — Investigate an Unexpected Restart

### Objective

Learn how analysts correlate events around a system restart.

### Steps

1. Open Windows Logs → System.
2. Filter for Event IDs `41,1001,1074,6008`.
3. Identify the approximate restart time.
4. Check whether a planned restart event (1074) exists.
5. Look for Event ID 41 or 6008, which may indicate an unexpected shutdown.
6. If Event ID 1001 exists, examine the bugcheck information.
7. Review nearby events for updates, driver changes, or other errors.
8. Document a conclusion based on the evidence.

Do not deliberately crash or interrupt power to your computer to generate these events.

### Interpretation

- Event ID 1074 may identify a process or user that initiated a restart.
- Event ID 41 indicates Windows did not shut down cleanly.
- Event ID 6008 records that the previous shutdown was unexpected.
- Event ID 1001 may contain bugcheck details.

These events are investigative clues. Event ID 41 or 6008 alone does not identify the root cause.

## 9. PowerShell: Read System Events

Open PowerShell. The following command reads recent System log events without changing system configuration.

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Id = 41, 1074, 6008, 7045
} -MaxEvents 30 |
Select-Object TimeCreated, ProviderName, Id, LevelDisplayName, Message |
Format-List
```

### Export a small investigation sample

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Id = 41, 1074, 6008, 7045
} -MaxEvents 100 |
Select-Object TimeCreated, ProviderName, Id, LevelDisplayName, Message |
Export-Csv "$env:USERPROFILE\Desktop\System-Events.csv" -NoTypeInformation
```

The CSV is saved to your Desktop. Review it before sharing because event messages can contain hostnames, account names, and file paths.

If no matching events exist, PowerShell may report that no events were found. That is not necessarily an error in your configuration.

## 10. SIEM Analysis with Splunk

Use these searches after System events have been ingested. Adjust the index and field names to match your environment.

### Search A — Review System Events

```spl
index=windows sourcetype="WinEventLog:System"
| table _time host source EventCode Message
| sort - _time
```

### Search B — Investigate Restart-Related Events

```spl
index=windows sourcetype="WinEventLog:System"
(EventCode=41 OR EventCode=1074 OR EventCode=6008)
| table _time host source EventCode Message
| sort - _time
```

### Search C — Review Service Installation Events

```spl
index=windows sourcetype="WinEventLog:System" EventCode=7045
| table _time host Message
| sort - _time
```

**Configuration note:** These examples assume the System log is ingested with the specified sourcetype and EventCode field. Your Splunk deployment may use another sourcetype, field name, or index. Verify actual events before relying on these searches.

## 11. SOC Investigation Checklist

- [ ] Identify the provider and Event ID.
- [ ] Record the timestamp and host.
- [ ] Review the event message and relevant details.
- [ ] Determine whether the activity was expected.
- [ ] Correlate related events around the same time.
- [ ] Review service installation details when relevant.
- [ ] Distinguish a symptom from a confirmed root cause.
- [ ] Record evidence and recommended follow-up.

## 12. Evidence and Screenshots



## 13. Key Takeaways

- System logs provide visibility into operating system and service activity.
- Restart-related events can help reconstruct a system timeline.
- Service installation events deserve context-based review.
- Correlation across multiple events is more reliable than interpreting a single Event ID.
- Splunk can centralize these records for searching, monitoring, and investigation.

## 14. References

- Microsoft Learn — [Troubleshoot unexpected reboots using system event logs](https://learn.microsoft.com/en-us/troubleshoot/windows-server/performance/troubleshoot-unexpected-reboots-system-event-logs)
- Microsoft Learn — [Event ID 41: The system has rebooted without cleanly shutting down first](https://learn.microsoft.com/en-us/troubleshoot/windows-client/performance/event-id-41-restart)
- Splunk Documentation — [Search Manual](https://docs.splunk.com/Documentation/Splunk/latest/Search/Aboutthesearchlanguage)

---

**Project:** SOC Sentinel — SIEM-Based Log Monitoring & Security Incident Investigation Lab

**Focus:** Windows System Logs | Event Viewer | PowerShell | Splunk | System Monitoring | SOC L1 Investigation
