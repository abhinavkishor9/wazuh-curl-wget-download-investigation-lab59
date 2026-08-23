# Timeline — Lab 59

## Curl / Wget Download Investigation

---

## Timeline Summary

| Time | Source | Activity | Significance |
|---|---|---|---|
| 07:21:16 | PowerShell | Hostname, user and timestamp identified | Investigation baseline established |
| 07:22 | PowerShell | `C:\CurlWgetDownloadLab` created | Investigation workspace established |
| 07:23:53 | PowerShell | Filesystem baseline created | Initial directory state recorded |
| 07:24 | PowerShell | `C:\CurlDownloadServer` created | Controlled download source prepared |
| 07:24 | PowerShell | Benign test file created | Expected download content established |
| ~07:25 | PowerShell | `ipconfig` executed | Network environment identified |
| ~07:26 | PowerShell | `curl.exe` located | Native Windows curl verified |
| ~07:26 | PowerShell | `curl.exe --version` | curl 8.13.0 confirmed |
| 07:28:10 | PowerShell | curl download initiated | Controlled download activity started |
| 07:29:10 | Sysmon | Event ID 1 recorded | curl.exe process execution confirmed |
| 07:29:10 | Filesystem | Output file created | 334-byte artifact observed |
| 07:29+ | PowerShell | File content inspected | HTTP 400 response identified |
| 07:30:38 | Event Viewer | Sysmon Event ID 1 reviewed | Process telemetry verified |
| 07:34:41 | PowerShell | Timestamp captured | Post-test time recorded |
| ~07:35 | Wazuh | Agent 001 queried | Windows Wazuh agent confirmed active |

---

# 1. 07:21:16 — Investigation Start

The initial time was recorded:

```text
23 August 2026 07:21:16
```

The endpoint was identified as:

```text
DESKTOP-9MMM37V
```

The logged-in user was:

```text
desktop-9mmm37v\dell
```

This established the initial investigation context.

---

# 2. 07:22 — Investigation Directory Created

The primary working directory was created:

```text
C:\CurlWgetDownloadLab
```

This directory was used as the destination for the controlled curl download.

---

# 3. 07:23:53 — Filesystem Baseline Created

The contents of the investigation directory were exported:

```powershell
Get-ChildItem "C:\CurlWgetDownloadLab" -Force |
Select-Object Name, Length, CreationTime, LastWriteTime |
Out-File "C:\CurlWgetDownloadLab\download-baseline.txt"
```

The baseline provided a reference point for identifying files created during the investigation.

---

# 4. 07:24 — Controlled Download Source Prepared

A second directory was created:

```text
C:\CurlDownloadServer
```

A known benign file was created:

```text
benign-download.txt
```

Expected content:

```text
This is a benign file used for Curl download investigation.
```

This established the expected content before the download test.

---

# 5. ~07:25 — Network Environment Checked

The network configuration was examined using:

```powershell
ipconfig
```

The relevant VMware adapter was:

```text
VMware Network Adapter VMnet1

IPv4:
192.168.174.1

Subnet:
255.255.255.0
```

Several physical and wireless adapters were disconnected.

---

# 6. ~07:26 — curl Binary Identified

The installed curl executable was identified:

```powershell
Get-Command curl.exe
```

Result:

```text
C:\Windows\System32\curl.exe
```

The version was checked:

```powershell
curl.exe --version
```

Result:

```text
curl 8.13.0
libcurl/8.13.0
```

This confirmed the use of the native Windows curl binary.

---

# 7. 07:28:10 — Download Activity Initiated

The controlled download command was executed:

```powershell
curl.exe http://127.0.0 -o C:\CurlWgetDownloadLab\benign-download.txt
```

Curl reported:

```text
100   334
```

This indicated that 334 bytes were transferred to the specified output file.

---

# 8. 07:29:10 — curl Process Creation Observed

Sysmon recorded Event ID 1:

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

User:
SYSTEM
```

This is the primary endpoint telemetry obtained during the investigation.

---

# 9. 07:29:10 — Output File Created

The expected output file was found:

```text
C:\CurlWgetDownloadLab\benign-download.txt
```

Metadata:

```text
Length:
334 bytes

CreationTime:
23-08-2026 07:29:10

LastWriteTime:
23-08-2026 07:29:10
```

The file creation was therefore confirmed through filesystem inspection.

---

# 10. 07:29+ — File Content Investigated

The downloaded file was inspected:

```powershell
Get-Content "C:\CurlWgetDownloadLab\benign-download.txt"
```

The file contained an HTML HTTP 400 response:

```text
Bad Request - Invalid Hostname
```

Therefore, the actual download result was:

```text
Expected:
Benign test file

Actual:
HTTP 400 error page
```

This established that the intended benign content was not retrieved.

---

# 11. 07:30:38 — Sysmon Event Review

The Sysmon Operational log was reviewed.

The relevant Event ID 1 entry showed the curl process.

This confirmed that process creation telemetry was available for the activity.

---

# 12. 07:34:41 — Post-Test Timestamp

The current time was captured:

```text
23 August 2026 07:34:41
```

This provided a clear endpoint for the main download investigation activity.

---

# 13. ~07:35 — Wazuh Agent Verification

The Wazuh manager was queried:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

The Windows endpoint was returned as:

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

This confirmed that the Windows endpoint was actively connected to Wazuh.

---

# 14. Telemetry Coverage

## Confirmed

```text
Windows Endpoint
        ↓
curl.exe execution
        ↓
Sysmon Event ID 1
        ↓
Output file creation
        ↓
File content inspection
        ↓
Wazuh agent confirmed active
```

## Not Obtained

```text
Sysmon Event ID 3
Network Connection

Sysmon Event ID 11
File Create

Windows Security Event ID 4688
Process Creation

PowerShell Event ID 4104
Script Block Logging
```

These events should be documented as **not obtained**, not as definitively absent.

---

# 15. Timeline Correlation

The most important sequence from the investigation is:

```text
07:28:10
        |
        v
curl download initiated
        |
        v
07:29:10
        |
        +---- Sysmon Event ID 1
        |
        +---- curl.exe process creation
        |
        +---- 334-byte output file
        |
        v
File content inspected
        |
        v
HTTP 400 error response identified
        |
        v
~07:35
Wazuh agent confirmed Active
```

---

# 16. Investigation Timeline Conclusion

The timeline confirms that the controlled curl download was initiated at approximately 07:28:10 and that `curl.exe` process creation was recorded by Sysmon Event ID 1 at approximately 07:29:10. A 334-byte output file was created at the same approximate time, but its content was an HTTP 400 error response rather than the intended benign file. The Wazuh agent was confirmed to be active, but additional network, file-creation, Windows Security, and PowerShell telemetry was not obtained during this session.

---

# 17. Final Timeline Assessment

**Primary confirmed activity:**

```text
curl.exe
    ↓
Download attempt
    ↓
334-byte file
    ↓
HTTP 400 response
    ↓
Sysmon Event ID 1
```

**Investigation status:**

```text
Completed
with telemetry limitations
```
