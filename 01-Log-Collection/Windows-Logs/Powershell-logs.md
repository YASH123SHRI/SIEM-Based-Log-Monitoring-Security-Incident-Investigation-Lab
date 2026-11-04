
# Windows PowerShell Logs – Collection and Analysis

## 1. Objective

The objective of this lab is to understand how Windows records PowerShell activity and how a SOC Analyst can use these logs to investigate potentially suspicious command execution.

PowerShell is a legitimate Windows administration tool. However, attackers may also misuse it to execute unwanted commands, modify system configurations, or perform other malicious activities.

In this lab, we will:

- Locate Windows PowerShell event logs.
- Understand important PowerShell Event IDs.
- Enable PowerShell Script Block Logging.
- Generate harmless PowerShell activity.
- Collect and examine PowerShell events.
- Use PowerShell commands to retrieve logs.
- Understand how PowerShell logs can be investigated in Splunk.
- Document findings in a SOC investigation format.

## 2. Lab Environment

| Component | Description |
|---|---|
| Operating System | Windows 11 |
| Log Collection Tool | Windows Event Viewer |
| Command-Line Tool | Windows PowerShell |
| SIEM | Wazuh (optional for this stage) |
| Log Source | Microsoft-Windows-PowerShell/Operational |
| Administrator Access | Required for some logging configuration changes |

## 3. Understanding PowerShell Logging

Windows PowerShell maintains operational logs that record information about PowerShell engine activity, commands, and script execution.

The primary log channel used in this lab is:

`Microsoft-Windows-PowerShell/Operational`

### Important PowerShell Event IDs

| Event ID | Description | SOC Relevance |
|---|---|---|
| 400 | PowerShell engine state changed to Available | Helps identify PowerShell session initialization |
| 403 | PowerShell engine state changed to Stopped | Helps identify session termination |
| 600 | Provider lifecycle event | Provides information about PowerShell providers |
| 4103 | Module logging information | Helps investigate command and module activity |
| 4104 | Script Block Logging | Records script block content when enabled |
| 4105 | Script block started execution | Helps identify script execution activity when enabled |
| 4106 | Script block completed execution | Helps identify script execution completion when enabled |

**Important:** Event IDs must be interpreted with their provider, event details, timestamp, and surrounding activity. A PowerShell event does not automatically indicate malicious activity.

## 4. Enabling PowerShell Logging

PowerShell Script Block Logging is useful because it records the content of processed script blocks in the PowerShell Operational log.

### Step 1: Open Local Group Policy Editor

1. Press `Win + R`.
2. Type `gpedit.msc`.
3. Press Enter.
4. Navigate to:

`Computer Configuration → Administrative Templates → Windows Components → Windows PowerShell`

5. Locate **Turn on PowerShell Script Block Logging**.
6. Double-click the setting.
7. Select **Enabled**.
8. Click Apply and OK.

If the setting offers an option to log script block invocation start and stop events, leave it disabled for this introductory lab.

### Step 2: Enable Module Logging (Optional)

1. In the same Windows PowerShell policy folder, locate **Turn on Module Logging**.
2. Open the policy.
3. Select **Enabled**.
4. Under Options, enable logging for all modules using `*`.
5. Click Apply and OK.

Module logging can generate additional events. It is optional for this introductory exercise.

### Step 3: Update Group Policy

Open Command Prompt or PowerShell as Administrator and run:

```powershell
gpupdate /force
```

Wait for the policy update to complete.

### Important Note for Windows Home Users

If `gpedit.msc` is unavailable, your Windows edition may not include Local Group Policy Editor. Do not download unofficial policy-editor packages.

You can complete the basic log exploration and harmless command-generation exercises without enabling additional policies. Script Block Logging may need to be enabled through an approved administrative configuration method.

## 5. Locating PowerShell Logs in Event Viewer

1. Press `Win + R`.
2. Type:

```text
eventvwr.msc
```

3. Press Enter.
4. Expand **Applications and Services Logs**.
5. Expand **Microsoft**.
6. Expand **Windows**.
7. Locate and select **PowerShell**.
8. Click **Operational**.

You should now see PowerShell-related events.

Record the following details from the events:

- Date and Time
- Event ID
- Level
- Source / Provider
- User
- Event details
- Script block text, if available

## 6. Practical Lab – Generate PowerShell Activity

We will generate harmless PowerShell activity to understand how it appears in Windows logs.

### Exercise 1: Start a PowerShell Session

1. Press `Win + X`.
2. Select Terminal or Windows PowerShell.
3. Open a PowerShell tab.
4. Run:

```powershell
$PSVersionTable
```

This displays the installed PowerShell version and environment information.

Now run:

```powershell
Get-Date
```

This displays the current date and time.

### Exercise 2: Generate Basic Command Activity

Run these commands one at a time:

```powershell
Get-Process
```

```powershell
Get-Service
```

```powershell
Get-ComputerInfo
```

```powershell
Get-NetIPAddress
```

These commands retrieve information about processes, services, the computer, and network interfaces.

They are read-only discovery commands and should not modify your system configuration.

### Exercise 3: Generate a Script Block Event

1. Open PowerShell.
2. Run the following harmless script:

```powershell
$computerName = $env:COMPUTERNAME
$currentUser = $env:USERNAME
$currentTime = Get-Date

Write-Output "Computer: $computerName"
Write-Output "User: $currentUser"
Write-Output "Time: $currentTime"
```

3. Return to Event Viewer.
4. Open:

`Applications and Services Logs → Microsoft → Windows → PowerShell → Operational`

5. Look for Event ID **4104**.
6. Open the event.
7. Navigate to the General tab.
8. Inspect the script block text.

If Script Block Logging was successfully enabled, you should be able to find the recorded script block.

**Note:** Events may take a short time to appear. If you cannot find Event ID 4104, verify that Script Block Logging is enabled and repeat the harmless script exercise.

### Exercise 4: Create and Execute a PowerShell Script File

1. Open Notepad.
2. Paste the following code:

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

6. Execute the script:

```powershell
.\SOC-PowerShell-Lab.ps1
```

If Windows blocks execution because of the current execution policy, do not change the system-wide policy just for this exercise. You can instead run the individual commands directly in PowerShell and inspect their logs.

7. Return to Event Viewer.
8. Look for Event ID 4104 and inspect the script content.

## 7. Retrieving PowerShell Logs Using PowerShell

A SOC Analyst should be comfortable collecting event logs directly through PowerShell.

### Exercise 1: Retrieve Recent PowerShell Events

Open PowerShell and execute:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 20 |
Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
Format-List
```

This command retrieves the latest 20 events from the PowerShell Operational log.

### Exercise 2: Retrieve Specific Event IDs

To retrieve Script Block Logging events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} -MaxEvents 20 |
Select-Object TimeCreated, Id, Message |
Format-List
```

To retrieve module logging events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4103
} -MaxEvents 20 |
Select-Object TimeCreated, Id, Message |
Format-List
```

If no results appear, the relevant event may not have been generated or the corresponding logging policy may not be enabled.

### Exercise 3: Export PowerShell Logs to CSV

Execute:

```powershell
$events = Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 100

$events |
Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
Export-Csv "$env:USERPROFILE\Desktop\PowerShell-Logs.csv" -NoTypeInformation
```

Open your Desktop and locate:

`PowerShell-Logs.csv`

You can open it in Excel to review the collected events.

**Handling note:** PowerShell logs may contain usernames, system details, command arguments, or other sensitive information. Review and redact sensitive information before uploading logs or screenshots to a public GitHub repository.

## 8. Basic PowerShell Log Analysis in Splunk

If your Windows logs are being forwarded to Splunk, you can use Search Processing Language (SPL) to investigate PowerShell activity.

The following examples assume:

- Index: `windows`
- Sourcetype: `WinEventLog:Microsoft-Windows-PowerShell/Operational`
- Event ID field: `EventCode`

Actual index, sourcetype, and field names depend on your Splunk configuration.

### Search 1: View PowerShell Events

```spl
index=windows sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational"
| table _time host EventCode Message
| sort - _time
```

### Search 2: Find Script Block Logging Events

```spl
index=windows EventCode=4104
| table _time host user Message
| sort - _time
```

### Search 3: Count PowerShell Events by Event ID

```spl
index=windows sourcetype="WinEventLog:Microsoft-Windows-PowerShell/Operational"
| stats count by EventCode
| sort - count
```

### Search 4: Review Script Block Activity Over Time

```spl
index=windows EventCode=4104
| timechart span=1h count
```

### Search 5: Review PowerShell Activity by Host

```spl
index=windows EventCode=4104
| stats count by host
| sort - count
```

**Note:** These searches are starting examples. Confirm your actual field names and data format using Splunk's Search interface before relying on them.

## 9. SOC Investigation – What Should an Analyst Look For?

When reviewing PowerShell logs, a SOC Analyst should investigate context rather than treating every command as suspicious.

Examples of useful investigation questions:

| Observation | Investigation Question |
|---|---|
| Unexpected PowerShell session | Which user started the session? |
| PowerShell launched by another application | Which parent process initiated it? |
| Script block contains unfamiliar commands | What does the script attempt to do? |
| Unusual execution time | Was the activity expected? |
| Repeated PowerShell activity | Is it associated with a legitimate administrative task? |
| PowerShell activity on an unexpected host | Is the host authorized to perform this task? |
| Script execution followed by configuration changes | Are the events related? |

### Important Investigation Considerations

- Check the account associated with the activity.
- Examine the event timestamp.
- Review the complete script block where available.
- Correlate PowerShell events with Windows Security and System logs.
- Check whether the activity is consistent with normal administrative work.
- Avoid classifying an event as malicious based only on a command name.

## 10. Practical SOC Investigation Exercise

### Scenario: Unusual PowerShell Script Execution

**Scenario Description:**

A monitoring system generates an alert indicating that PowerShell activity occurred on a Windows workstation outside the user's expected working period.

As a SOC Analyst, your task is to review the available logs and determine whether the activity appears legitimate or requires further investigation.

### Investigation Steps

1. Open Event Viewer.
2. Navigate to the PowerShell Operational log.
3. Filter for Event ID 4104.
4. Identify the timestamp of the event.
5. Review the script block content.
6. Identify the user and computer involved, if available.
7. Check nearby PowerShell engine events such as 400 and 403.
8. Correlate the timestamp with Windows Security logs.
9. Record your observations.
10. State whether the evidence is sufficient to classify the activity.

Do not assume that activity outside normal working hours is malicious. A legitimate administrator or scheduled task may generate PowerShell activity.

### Investigation Report Template

```text
Incident Title: Unusual PowerShell Activity

Date and Time:
Hostname:
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

## 11. Screenshots to Capture for GitHub

Save relevant screenshots under:

| Screenshot | What to Capture |
|---|---|
| 01-PowerShell-Operational-Log.png | PowerShell Operational log in Event Viewer |
| 02-Script-Block-Logging-Policy.png | Enabled Script Block Logging policy |
| 03-PowerShell-Commands.png | Harmless PowerShell commands being executed |
| 04-Event-ID-4104.png | Script Block Logging event |
| 05-Event-ID-4103.png | Module logging event, if enabled and generated |
| 06-Get-WinEvent-Output.png | PowerShell command retrieving event logs |
| 07-PowerShell-CSV-Export.png | Exported CSV file |
| 08-Splunk-PowerShell-Search.png | PowerShell events in Splunk, if configured |

<img width="1302" height="487" alt="image" src="https://github.com/user-attachments/assets/0260b2d8-def7-40c1-83b7-7d2829503ec7" />

## 12. Key Takeaways

After completing this lab, you should understand:

- Where Windows stores PowerShell Operational events.
- How to enable Script Block Logging.
- The purpose of important PowerShell Event IDs.
- How to generate and identify harmless PowerShell activity.
- How to retrieve logs using Get-WinEvent.
- How to export logs for further investigation.
- How to begin analysing PowerShell activity using Splunk.
- How to document PowerShell-related findings in a SOC investigation report.

## 13. References

- Microsoft Learn – PowerShell Logging: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging
- Microsoft Learn – Group Policy Settings for PowerShell: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_group_policy_settings
- Microsoft Learn – Get-WinEvent: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent
- Splunk Documentation: https://docs.splunk.com/

---

**Project:** SOC Sentinel – SIEM-Based Log Monitoring & Security Incident Investigation Lab

**Section:** 02 – Log Collection

**Category:** Windows Logs

**Status:** Complete this documentation only after performing and recording the corresponding lab activities.
