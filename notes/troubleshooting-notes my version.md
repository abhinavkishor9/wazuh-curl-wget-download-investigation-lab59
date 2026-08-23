# Troubleshooting Notes 

## 1. Purpose

This document records the troubleshooting observations and telemetry limitations encountered during Lab 59.

The main issue was not failure of the curl command itself. The curl process executed and created an output file. The primary limitation was incomplete endpoint telemetry.

Only **Sysmon Event ID 1** was successfully obtained.

---

# 2. Download Returned HTTP 400

## Observation

The controlled curl command was:

```powershell
curl.exe http://127.0.0 -o C:\CurlWgetDownloadLab\benign-download.txt
```

Curl reported:

```text
100   334
```

However, inspecting the resulting file showed:

```text
Bad Request - Invalid Hostname
```

The file was therefore an HTTP 400 HTML response.

---

## Interpretation

The transfer itself completed, but the expected benign content was not retrieved.

The result should be recorded as:

```text
Transfer:
Completed

Output file:
Created

Expected benign content:
Not retrieved

Actual content:
HTTP 400 error page
```

---

## Lesson

Do not assume that a successful transfer percentage means that the intended payload was downloaded.

Always verify the resulting file.

Useful commands include:

```powershell
Get-Item "C:\CurlWgetDownloadLab\benign-download.txt"
Get-Content "C:\CurlWgetDownloadLab\benign-download.txt"
Get-FileHash "C:\CurlWgetDownloadLab\benign-download.txt"
```

---

# 3. Sysmon Event ID 1 Was Available

Sysmon Event ID 1 was successfully located.

Relevant evidence:

```text
Event ID:
1

Task Category:
Process Create

Image:
C:\WINDOWS\System32\curl.exe

FileVersion:
8.13.0

Process ID:
18076

Computer:
DESKTOP-9MMM37V
```

This confirms that the curl process was executed.

---

# 4. Sysmon Event ID 3 Was Not Obtained

## Expected Event

```text
Sysmon Event ID 3
Network Connection
```

## Result

No usable Event ID 3 evidence was obtained during the lab.

## Possible Reasons

Possible explanations include:

- Event ID 3 may not be enabled in the active Sysmon configuration.
- The event may have been filtered.
- Wazuh may not be collecting the relevant Sysmon event.
- The event may have occurred outside the search window.
- The lab network configuration may affect visibility.

The available evidence is not sufficient to determine the exact reason.

---

## Recommended Check

Review the active Sysmon configuration and determine whether NetworkConnect monitoring is enabled.

The investigation should verify the actual active configuration rather than assuming that all Sysmon event types are enabled.

---

# 5. Sysmon Event ID 11 Was Not Obtained

## Expected Event

```text
Sysmon Event ID 11
File Create
```

## Result

The file was definitely created because PowerShell returned:

```text
C:\CurlWgetDownloadLab\benign-download.txt
```

with:

```text
Length:
334 bytes
```

However, a corresponding Sysmon Event ID 11 was not obtained.

---

## Important Distinction

The evidence establishes:

```text
File creation:
Confirmed
```

It does not establish:

```text
Sysmon Event ID 11:
Generated
```

or:

```text
Sysmon Event ID 11:
Not generated
```

The correct statement is:

> Sysmon Event ID 11 was not obtained during the investigation.

---

# 6. Windows Security Event ID 4688 Was Not Obtained

## Expected Event

```text
Windows Security Event ID 4688
A new process has been created
```

## Result

Event ID 4688 was not obtained during this lab.

Possible reasons include:

- Process Creation auditing is not enabled.
- Command-line auditing is not enabled.
- The Security log was not being collected.
- Wazuh was not configured to collect the relevant event.
- Event filtering may be present.

---

## Recommended Validation

Check:

```text
Advanced Audit Policy Configuration
    >
Detailed Tracking
    >
Audit Process Creation
```

Also verify that command-line information is being recorded when required.

---

# 7. PowerShell Event ID 4104 Was Not Obtained

## Expected Event

```text
Microsoft-Windows-PowerShell/Operational
Event ID 4104
Script Block Logging
```

## Result

No usable Event ID 4104 evidence was obtained.

Possible reasons include:

- Script Block Logging is disabled.
- The PowerShell Operational channel is not being collected.
- Wazuh is not configured to collect the channel.
- Event filtering is present.

---

## Recommended Validation

Verify that the following channel is available:

```text
Microsoft-Windows-PowerShell/Operational
```

Then confirm whether Script Block Logging is enabled.

---

# 8. Wazuh Agent Was Active

The Wazuh manager was checked using:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

The result showed:

```text
Agent ID:
001

Agent Name:
DESKTOP-9MMM37V

Status:
Active

Operating System:
Microsoft Windows 11 Pro

Client Version:
Wazuh v4.12.0
```

Therefore, the Windows endpoint was connected to Wazuh.

---

# 9. Active Agent Does Not Mean Complete Telemetry

An important troubleshooting lesson is that:

```text
Wazuh Agent:
Active
```

does not automatically mean:

```text
All Windows event channels:
Collected
```

Wazuh requires appropriate configuration to collect and forward the relevant Windows event sources.

Therefore, missing Event IDs should be investigated at the telemetry configuration level.

---

# 10. Network Environment

The endpoint was operating with VMware networking.

The relevant adapter was:

```text
VMware Network Adapter VMnet1

IPv4:
192.168.174.1

Subnet:
255.255.255.0
```

Several other adapters were disconnected.

When troubleshooting network telemetry, the complete path should be considered:

```text
Windows Endpoint
        |
        v
VMware Network Adapter
        |
        v
Virtual Network
        |
        v
Host / Lab Network
```

Network architecture can affect what network telemetry is visible at the endpoint and SIEM.

---

# 11. curl Binary Verification

The installed binary was confirmed with:

```powershell
Get-Command curl.exe
```

Result:

```text
C:\Windows\System32\curl.exe
```

Version:

```text
curl 8.13.0
```

This was useful because the executable path itself is an important detection attribute.

For example:

```text
C:\Windows\System32\curl.exe
```

would generally require different investigation context from:

```text
C:\Users\<user>\AppData\Local\Temp\curl.exe
```

The binary location should therefore be considered when building detections.

---

# 12. Recommended Telemetry Validation

Before repeating the download test, validate the following:

| Telemetry | Event | Purpose |
|---|---:|---|
| Sysmon | 1 | Process Creation |
| Sysmon | 3 | Network Connection |
| Sysmon | 11 | File Creation |
| Windows Security | 4688 | Process Creation |
| PowerShell | 4104 | Script Block Logging |

Then perform the download again and search within a narrow time range around the activity.

---

# 13. Recommended Troubleshooting Sequence

Use the following sequence for the next iteration:

```text
1. Verify Sysmon service
        ↓
2. Verify active Sysmon configuration
        ↓
3. Confirm Event ID 1
        ↓
4. Confirm Event ID 3 configuration
        ↓
5. Confirm Event ID 11 configuration
        ↓
6. Verify Windows Process Creation auditing
        ↓
7. Verify PowerShell Script Block Logging
        ↓
8. Verify Wazuh Windows event collection
        ↓
9. Repeat controlled curl activity
        ↓
10. Search logs using the exact activity timestamp
        ↓
11. Correlate process, network, and file events
```

---

# 14. Recommended Investigation Method

Do not attempt to troubleshoot every telemetry source simultaneously.

Use a layered approach:

```text
Layer 1
Was curl.exe executed?
        ↓
Sysmon Event ID 1

Layer 2
Did curl make a network connection?
        ↓
Sysmon Event ID 3

Layer 3
Was a file created?
        ↓
Sysmon Event ID 11

Layer 4
Was the process also visible in Windows Security?
        ↓
Event ID 4688

Layer 5
Was the PowerShell command/script captured?
        ↓
Event ID 4104

Layer 6
Did Wazuh receive the endpoint telemetry?
        ↓
Wazuh analysis
```

This approach makes it easier to identify exactly where telemetry is being lost.

---

