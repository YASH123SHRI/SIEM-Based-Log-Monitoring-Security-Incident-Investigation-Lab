
# Windows PowerShell Logs – Collection and Analysis Using Wazuh

## 1. Objective

The objective of this lab is to configure Windows PowerShell logging, collect PowerShell events through the Wazuh Agent, and investigate the collected events using the Wazuh Dashboard.

PowerShell is a legitimate Windows administration tool. However, it can also be misused to perform unauthorized activities. Monitoring PowerShell activity helps SOC Analysts understand command execution and investigate potentially suspicious behaviour.

In this lab, we will:

- Understand Windows PowerShell logging.
- Enable PowerShell Script Block Logging.
- Generate harmless PowerShell activity.
- Configure the Wazuh Agent to collect PowerShell Operational events.
- Verify communication between the Windows endpoint and Wazuh Manager.
- Investigate collected events in the Wazuh Dashboard.
- Retrieve and export PowerShell logs.
- Document findings in a SOC investigation report.

## 2. Lab Environment

| Component | Description |
|---|---|
| Endpoint | Windows 11 |
| SIEM | Wazuh |
| Log Collection | Wazuh Agent |
| Log Source | Microsoft-Windows-PowerShell/Operational |
| Log Analysis | Wazuh Dashboard |
| Configuration | Windows Event Viewer and Wazuh Agent `ossec.conf` |
| Additional Tool | Windows PowerShell |

### Lab Architecture

```text
+---------------------------+
|       Windows 11          |
|                           |
|  PowerShell Commands      |
|  Script Execution         |
|           |               |
|           v               |
|  PowerShell Operational   |
|  Event Log                |
|           |               |
|           v               |
|      Wazuh Agent          |
+-----------|---------------+
            |
            | Event Log Forwarding
            |
            v
+---------------------------+
|       Wazuh Manager       |
|                           |
|  Log Collection           |
|  Decoders                 |
|  Detection Rules          |
+-----------|---------------+
            |
            v
+---------------------------+
|      Wazuh Dashboard      |
|                           |
|  Threat Hunting           |
|  Event Investigation      |
|  Alert Analysis           |
+---------------------------+
```

## 3. Understanding PowerShell Logging

Windows PowerShell records operational information in a dedicated Windows Event Log channel.

The primary channel used in this experiment is:

`Microsoft-Windows-PowerShell/Operational`

This channel can contain information about PowerShell engine activity, module activity, and script block execution.

### Important PowerShell Event IDs

| Event ID | Description | SOC Relevance |
|---|---|---|
| 400 | PowerShell engine state changed to Available | Helps identify PowerShell session initialization |
| 403 | PowerShell engine state changed to Stopped | Helps identify session termination |
| 4103 | Module logging information | Helps investigate command and module activity |
| 4104 | Script Block Logging | Records processed script block content when enabled |
| 4105 | Script block started execution | Available when script block invocation logging is enabled |
| 4106 | Script block completed execution | Available when script block invocation logging is enabled |

**Important:** Event IDs must be interpreted alongside their provider, event details, timestamp, user, and surrounding activity. A PowerShell event does not automatically indicate malicious activity.

## 4. Step-by-Step Lab Execution

### Step 1: Verify the Wazuh Agent

Before configuring PowerShell logging, confirm that your Windows machine is connected to your Wazuh Manager.

1. Open the Wazuh Dashboard.
2. Navigate to **Agents**.
3. Locate your Windows 11 endpoint.
4. Check that its status is **Active**.
5. Note the agent name and agent ID.

Record these details:

| Field | Value |
|---|---|
| Agent Name | |
| Agent ID | |
| Operating System | Windows 11 |
| Agent Status | Active / Not Active |
| Wazuh Manager | |

If the agent is not active, resolve the connection issue before continuing.

### Step 2: Enable PowerShell Script Block Logging

Script Block Logging records the content of PowerShell script blocks processed by the PowerShell engine.

1. Press `Win + R`.
2. Type:

```text
gpedit.msc
```

3. Press Enter.
4. Navigate to:

`Computer Configuration → Administrative Templates → Windows Components → Windows PowerShell`

5. Locate **Turn on PowerShell Script Block Logging**.
6. Double-click the setting.
7. Select **Enabled**.
8. Click Apply and OK.

For this introductory lab, leave the option to log script block invocation start and stop events disabled.

This keeps the experiment focused on Event ID 4104.

### Windows Home Edition

If `gpedit.msc` is unavailable, your Windows edition may not include Local Group Policy Editor.

You can still complete the basic PowerShell log exploration exercises. Script Block Logging must be enabled using an approved administrative configuration method before you expect Event ID 4104 to be generated.

### Step 3: Update Group Policy

Open PowerShell as Administrator and execute:

```powershell
gpupdate /force
```

Wait for the policy update to finish.

### Step 4: Locate PowerShell Operational Logs

1. Press `Win + R`.
2. Type:

```text
eventvwr.msc
```

3. Press Enter.
4. Navigate to:

`Applications and Services Logs → Microsoft → Windows → PowerShell → Operational`

5. Select **Operational**.

You should now be able to view PowerShell-related events.

At this stage, note the existing events and their timestamps.

### Step 5: Generate Harmless PowerShell Activity

Open a normal PowerShell window and execute the following commands individually.

**Command 1 – Display PowerShell version**

```powershell
$PSVersionTable
```

**Command 2 – Display current date and time**

```powershell
Get-Date
```

**Command 3 – Retrieve running processes**

```powershell
Get-Process
```

**Command 4 – Retrieve Windows services**

```powershell
Get-Service
```

**Command 5 – Retrieve network interface information**

```powershell
Get-NetIPAddress
```

These commands retrieve system information and do not intentionally modify system configuration.

### Step 6: Generate a Script Block Event

Execute the following harmless script in PowerShell:

```powershell
$computerName = $env:COMPUTERNAME
$currentUser = $env:USERNAME
$currentTime = Get-Date

Write-Output "Computer: $computerName"
Write-Output "User: $currentUser"
Write-Output "Time: $currentTime"
```

Return to Event Viewer and refresh the PowerShell Operational log.

Look for **Event ID 4104**.

Open the event and inspect:

- Date and Time
- Event ID
- User
- Script Block ID
- Script Block Text
- Message

You should be able to identify the script block that was generated by your test.

If Event ID 4104 does not appear:

1. Verify that Script Block Logging is enabled.
2. Run `gpupdate /force` again.
3. Execute the harmless script again.
4. Refresh Event Viewer.
5. Confirm that you are checking the correct PowerShell Operational channel.

### Step 7: Create and Execute a PowerShell Script File

1. Open Notepad.
2. Paste the following script:

```powershell
$computer = $env:COMPUTERNAME
$user = $env:USERNAME

Write-Output "SOC PowerShell Logging Lab"
Write-Output "Computer Name: $computer"
Write-Output "Logged-in User: $user"

Get-Date
Get-Process | Select-Object -First 5 ProcessName, Id
```

3. Save the file on your Desktop as:

`SOC-PowerShell-Lab.ps1`

4. Open PowerShell.
5. Navigate to your Desktop:

```powershell
Set-Location "$env:USERPROFILE\Desktop"
```

6. Execute:

```powershell
.\SOC-PowerShell-Lab.ps1
```

If Windows blocks script execution because of the current execution policy, do not change the system-wide policy just for this exercise. You can run the individual commands directly in PowerShell instead.

7. Return to Event Viewer.
8. Look for Event ID 4104.
9. Inspect the script block content.

## 5. Configure Wazuh Agent to Collect PowerShell Logs

This is the most important SIEM configuration step in this experiment.

The Wazuh Agent commonly monitors the Windows Application, Security, and System event channels by default. The PowerShell Operational channel must be added explicitly.

### Step 1: Locate the Wazuh Agent Configuration File

On your Windows 11 endpoint, open File Explorer.

Navigate to:

```text
C:\Program Files (x86)\ossec-agent\
```

Locate:

```text
ossec.conf
```

This is the Wazuh Agent configuration file.

### Step 2: Open the Configuration File

1. Right-click `ossec.conf`.
2. Open it using Notepad as Administrator.
3. Locate the main `<ossec_config>` section.

Do not delete or replace the existing configuration.

### Step 3: Add the PowerShell Event Channel

Add the following configuration block inside the existing `<ossec_config>` element:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Your configuration should contain the new block alongside the existing configuration sections.

**Important:**

- Do not create a second outer `<ossec_config>` element.
- Do not remove existing Security, Application, or System log collection settings.
- Ensure that the XML tags are correctly opened and closed.
- Save the configuration file.

### Step 4: Restart the Wazuh Agent

Open PowerShell as Administrator and execute:

```powershell
Restart-Service -Name wazuh
```

Check the service status:

```powershell
Get-Service -Name wazuh
```

Expected status:

```text
Status : Running
```

If the service fails to start, review the XML configuration for errors and inspect the Wazuh Agent logs before making further changes.

### Step 5: Verify the Configuration

After restarting the agent:

1. Open the Wazuh Dashboard.
2. Navigate to **Agents**.
3. Confirm that the Windows agent is Active.
4. Return to your Windows endpoint.
5. Generate the harmless PowerShell script activity again.
6. Wait briefly for the events to be forwarded and processed.

## 6. Verify PowerShell Events in Wazuh Dashboard

### Step 1: Open Threat Hunting

1. Log in to the Wazuh Dashboard.
2. Open **Threat Hunting**.
3. Select the Windows agent as the endpoint of interest, if the interface provides that filter.
4. Set the time range to cover the period when you executed your test script.

### Step 2: Search for PowerShell Activity

Search for events associated with the PowerShell Operational channel.

Depending on your Wazuh version and event mapping, useful fields may include:

- `agent.name`
- `agent.id`
- `win.system.channel`
- `win.system.eventID`
- `win.system.providerName`
- `win.eventdata.scriptBlockText`

For the 4104 event, inspect the event details and locate the script block text.

**Note:** Wazuh field names and available views can differ between versions. If a field is not available, open an event's expanded JSON details and identify the actual field names in your environment.

### Step 3: Understand Alerts Versus Collected Events

This distinction is important:

- **Collected events:** Events received by Wazuh from the endpoint.
- **Alerts:** Events that match Wazuh detection rules and satisfy the configured alert criteria.

Not every collected PowerShell event automatically becomes an alert.

If you cannot find your 4104 event in Threat Hunting, it may have been collected without triggering an alert.

To investigate all received events, Wazuh can also store event archives. Archive collection and indexing require additional configuration, described in Section 8.

## 7. Retrieve PowerShell Logs Locally

You can also retrieve PowerShell logs directly from Windows to compare them with the events received by Wazuh.

### Exercise 1: Retrieve Recent Events

Execute:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 20 |
Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
Format-List
```

### Exercise 2: Retrieve Event ID 4104

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} -MaxEvents 20 |
Select-Object TimeCreated, Id, Message |
Format-List
```

### Exercise 3: Retrieve Event ID 4103

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4103
} -MaxEvents 20 |
Select-Object TimeCreated, Id, Message |
Format-List
```

If no results appear for Event ID 4103, verify whether Module Logging is enabled and whether relevant activity has been generated.

### Exercise 4: Export PowerShell Logs to CSV

Execute:

```powershell
$events = Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 100

$events |
Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
Export-Csv "$env:USERPROFILE\Desktop\PowerShell-Logs.csv" -NoTypeInformation
```

Locate the exported file:

`Desktop\PowerShell-Logs.csv`

Open it in Excel and review the collected events.

Review and redact sensitive information before sharing any logs or screenshots publicly.

## 8. Optional: Enable Wazuh Event Archives

By default, Wazuh's alert output does not represent every event received from an endpoint.

Wazuh can store all received events in archive files, including events that do not trigger detection rules.

This is useful when investigating PowerShell Event ID 4104, because a normal script block may be collected without generating a security alert.

**This section requires administrative access to the Wazuh Manager.**

### Step 1: Open the Wazuh Manager Configuration

On the Wazuh Manager, open:

```text
/var/ossec/etc/ossec.conf
```

### Step 2: Locate the Global Configuration

Find the existing `<global>` section.

Inside that section, ensure the following settings are configured:

```xml
<global>
  <jsonout_output>yes</jsonout_output>
  <alerts_log>yes</alerts_log>
  <logall>no</logall>
  <logall_json>yes</logall_json>
</global>
```

If these entries already exist, update the relevant values rather than creating duplicate entries.

`logall_json` enables JSON archiving of all received events.

**Storage consideration:** Archives can consume significant disk space because they include events that do not generate alerts. Enable them only in your lab environment when you have sufficient storage and a suitable retention plan.

### Step 3: Restart Wazuh Manager

On a Linux-based Wazuh Manager, execute:

```bash
sudo systemctl restart wazuh-manager
```

Check the service:

```bash
sudo systemctl status wazuh-manager
```

### Step 4: Verify Archive Files

Check whether the archive directory contains the JSON event file:

```bash
sudo ls -lh /var/ossec/logs/archives/
```

The expected file is:

```text
/var/ossec/logs/archives/archives.json
```

Generate another harmless PowerShell script event on Windows and check whether it is received by the manager.

### Step 5: Access Archives in the Dashboard

Archive visualization requires the archive data to be indexed and an appropriate index pattern or data view to be available.

In a standard Wazuh setup, this may require additional Filebeat and Wazuh Indexer configuration.

Once configured:

1. Open the Wazuh Dashboard.
2. Navigate to **Explore → Discover**, where available.
3. Select the `wazuh-archives-*` data view.
4. Set the appropriate time range.
5. Filter for your Windows agent.
6. Inspect events from the PowerShell Operational channel.
7. Locate Event ID 4104 and examine the script block content.

If the archive data view is not available, do not assume that the agent has failed. First verify that `archives.json` contains the events, then check the archive indexing configuration.

## 9. SOC Investigation Exercise

### Scenario: Unusual PowerShell Activity

**Scenario Description:**

A SOC monitoring system identifies PowerShell activity on a Windows workstation outside the user's expected working period.

Your task is to use Wazuh and Windows Event Viewer to investigate the activity.

### Investigation Steps

1. Open Wazuh Dashboard.
2. Select the Windows endpoint.
3. Navigate to Threat Hunting or the configured archive data view.
4. Locate PowerShell Operational events.
5. Identify Event ID 4104.
6. Record the timestamp.
7. Review the script block content.
8. Identify the associated user and computer.
9. Examine nearby PowerShell engine events such as 400 and 403.
10. Correlate the activity with Windows Security and System logs.
11. Record your observations.
12. Decide whether the evidence is sufficient to classify the activity.

Do not assume that activity outside normal working hours is malicious. Scheduled tasks, administrators, and legitimate software can generate PowerShell activity.

### Investigation Report Template

```text
Incident Title: Unusual PowerShell Activity

Date and Time:
Hostname:
Agent ID:
Username:
Event ID:
Log Source:
Script Block Summary:

Observations:
1.
2.
3.

Related Events:
1.
2.

Investigation Findings:

Severity:
Low / Medium / High / Undetermined

Conclusion:

Recommended Actions:
1.
2.
3.
```

## 10. Screenshots to Capture for GitHub


| Screenshot | What to Capture |
|---|---|
| 01-PowerShell-Operational-Log.png | PowerShell Operational log in Event Viewer |
| 02-Script-Block-Logging-Policy.png | Enabled Script Block Logging policy |
| 03-PowerShell-Commands.png | Harmless PowerShell commands being executed |
| 04-Event-ID-4104.png | Script Block Logging event |
| 05-Wazuh-Agent-Configuration.png | PowerShell event channel configuration |
| 06-Wazuh-Agent-Status.png | Active Windows agent in Wazuh |
| 07-Wazuh-PowerShell-Events.png | PowerShell events visible in Wazuh |
| 08-Wazuh-Event-Details.png | Expanded event details |
| 09-PowerShell-CSV-Export.png | Exported CSV file |
| 10-Wazuh-Archives.png | Archived event, if archive collection is configured |

<img width="1302" height="418" alt="image" src="https://github.com/user-attachments/assets/1397125b-3258-403a-8b58-649545a54df2" />

## 11. Key Takeaways

After completing this lab, you should understand:

- How Windows records PowerShell activity.
- How to enable Script Block Logging.
- The purpose of important PowerShell Event IDs.
- How to generate harmless PowerShell activity.
- How to configure Wazuh Agent event-channel collection.
- How Windows events are forwarded to Wazuh Manager.
- How to verify endpoint activity in Wazuh.
- The difference between collected events and security alerts.
- Why event archives are useful during investigations.
- How to document PowerShell-related findings in a SOC report.

## 12. References

### Microsoft Documentation

- PowerShell Logging: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging
- Group Policy Settings for PowerShell: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_group_policy_settings
- Get-WinEvent: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent

### Wazuh Documentation

- Windows Log Collection: https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html
- Wazuh Agent Log Collection Configuration: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html
- Wazuh Log Data Collection Use Cases: https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/use-cases.html
- Wazuh Event Logging and Archives: https://documentation.wazuh.com/current/user-manual/manager/event-logging.html

---

**Project:** SOC Sentinel – SIEM-Based Log Monitoring & Security Incident Investigation Lab

**Section:** 02 – Log Collection

**Category:** Windows Logs

**SIEM:** Wazuh

**Status:** In Progress – Update after performing the practical exercises.
