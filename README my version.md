# wazuh-curl-wget-download-investigation-lab59
## Overview

curl and wget are command-line utilities used to retrieve content from remote servers.

A simple example is:

curl.exe → remote server → file downloaded to endpoint

An attacker can abuse legitimate download utilities to transfer a payload onto a compromised system.

This is why an analyst may investigate activity such as:

curl.exe http://server/file.exe -o file.exe

or:

wget http://server/file.exe

The important point is:

The use of curl or wget alone does not make the activity malicious.

They are legitimate tools. The analyst must investigate the complete context.

Imagine the following process chain:

powershell.exe
        ↓
cmd.exe
        ↓
curl.exe
        ↓
HTTP connection
        ↓
payload downloaded
        ↓
payload.exe
        ↓
payload executed

There are several investigation points.

Process evidence

Who launched curl.exe?

Command-line evidence

What URL was requested?

Network evidence

Which IP address or server was contacted?

File evidence

Where was the downloaded file saved?

Execution evidence

Was the downloaded file executed?

This is much more useful than simply seeing:

curl.exe executed

Modern Windows systems can contain:

C:\Windows\System32\curl.exe

This means the following command may be completely legitimate:

curl.exe --version

The presence of curl.exe is therefore not an alert by itself.

An analyst should ask:

Was curl.exe actually used to download something?
What was downloaded?
From where?
Where was the file saved?
Was the URL expected?
Was the downloaded file later executed?

Unlike curl.exe, wget.exe is not normally a standard built-in utility on Windows installations.

If wget.exe is observed, an analyst should investigate:

Where did wget.exe come from?
What is its full file path?
Is it part of approved software?
What is its hash?
Who executed it?
What did it download?

A download may deserve investigation when:

curl.exe is launched by an unusual parent process.
The destination is an unknown or suspicious server.
The downloaded file is an executable or script.
The file is saved in an unusual directory.
The downloaded file is immediately executed.
The command contains unexpected options.
The activity occurs under an unusual user account.
The download is inconsistent with the workstation's normal role.

Example:

winword.exe
    ↓
powershell.exe
    ↓
curl.exe
    ↓
payload.exe downloaded
    ↓
payload.exe executed

This would require significantly more investigation than a user downloading an approved document.


This lab investigates Windows command-line download activity using the native `curl.exe` utility and validates the resulting process activity through Sysmon and Wazuh telemetry.

The objective was to establish a filesystem baseline, prepare a controlled download scenario, execute a `curl.exe` download, verify the resulting file, and investigate the available endpoint telemetry.

The investigation successfully confirmed `curl.exe` execution through **Sysmon Event ID 1 (Process Creation)** and verified that the Windows endpoint was connected to Wazuh.

Only **Sysmon Event ID 1** was successfully obtained during this lab session. Other expected telemetry such as Sysmon Event ID 3, Sysmon Event ID 11, Windows Security Event ID 4688, and PowerShell Event ID 4104 was not obtained. Therefore, this lab does not claim complete download telemetry coverage.

---

## Repository Information

**Repository Name:**

`wazuh-curl-wget-download-investigation-lab59`

**Lab:**

Lab 59 — Curl / Wget Download Investigation

**Category:**

SOC / DFIR / Endpoint Investigation

**Investigation Date:**

23 August 2026

---

## Environment

| Component | Details |
|---|---|
| Operating System | Windows 11 Pro |
| Hostname | DESKTOP-9MMM37V |
| User | desktop-9mmm37v\dell |
| PowerShell | 7.6.5 |
| curl | 8.13.0 |
| curl Path | `C:\Windows\System32\curl.exe` |
| Wazuh Agent | 4.12.0 |
| Wazuh Agent ID | 001 |
| Wazuh Agent Status | Active |
| Endpoint Telemetry | Sysmon |
| Confirmed Sysmon Event | Event ID 1 |

---

## Investigation Objectives

- Investigate the execution of the legitimate Windows curl.exe utility in a controlled environment.
- Establish a filesystem baseline before download activity occurs.
- Observe and document the creation of a file as a result of a command-line download.
- Validate curl.exe execution using Sysmon Event ID 1.
- Examine the downloaded file to determine whether the expected content was actually retrieved.
- Verify the health and connectivity of the Windows endpoint's Wazuh agent.
- Identify gaps in available endpoint telemetry when investigating command-line download activity.
- Distinguish between confirmed evidence, missing telemetry, and assumptions during a SOC investigation.
- Assess how curl.exe download behavior could appear in a real-world attack chain.
- Identify additional telemetry that would be required for stronger process → network → file correlation.

---

# 1. Environment Baseline

The investigation started by identifying the Windows endpoint, logged-in user, and current timestamp.

```powershell
hostname
whoami
Get-Date
```

Observed:

```text
Hostname:
DESKTOP-9MMM37V

User:
desktop-9mmm37v\dell

Timestamp:
23 August 2026 07:21:16
```

A dedicated investigation directory was then created:

```powershell
New-Item -Path "C:\CurlWgetDownloadLab" -ItemType Directory -Force
```

The directory was verified using:

```powershell
Get-Item "C:\CurlWgetDownloadLab"
```

---

# 2. Filesystem Baseline

A baseline of the investigation directory was collected before performing the download:

```powershell
Get-ChildItem "C:\CurlWgetDownloadLab" -Force |
Select-Object Name, Length, CreationTime, LastWriteTime |
Out-File "C:\CurlWgetDownloadLab\download-baseline.txt"
```

The purpose of the baseline was to establish the initial state of the directory so that newly created files could be identified later.

---

# 3. Controlled Download Source

A second directory was created to represent the controlled download source:

```powershell
New-Item -Path "C:\CurlDownloadServer" -ItemType Directory -Force
```

A known benign test file was created:

```powershell
"This is a benign file used for Curl download investigation." |
Out-File "C:\CurlDownloadServer\benign-download.txt"
```

The intended content was therefore:

```text
This is a benign file used for Curl download investigation.
```

---

# 4. Network Environment

The Windows network configuration was checked using:

```powershell
ipconfig
```

The relevant virtual network adapter was:

```text
VMware Network Adapter VMnet1

IPv4 Address:
192.168.174.1

Subnet Mask:
255.255.255.0
```

Several physical and wireless adapters were shown as disconnected.

This confirmed that the lab was operating in a VMware-based virtual network environment.

---

# 5. curl Binary Verification

Before executing the download, the Windows `curl.exe` binary was verified.

```powershell
Get-Command curl.exe
```

The executable was located at:

```text
C:\Windows\System32\curl.exe
```

The version was then checked:

```powershell
curl.exe --version
```

Observed:

```text
curl 8.13.0
libcurl/8.13.0
```

This confirmed that the native Windows curl executable was being used.

---

# 6. Download Execution

At approximately 07:28:10, the controlled download activity was initiated.

The command used during the lab was:

```powershell
curl.exe http://127.0.0 -o C:\CurlWgetDownloadLab\benign-download.txt
```

The transfer output showed:

```text
100   334
```

A file was therefore created at the requested output location.

---

# 7. Downloaded File Verification

The resulting file was inspected:

```powershell
Get-Item "C:\CurlWgetDownloadLab\benign-download.txt"
```

Observed metadata:

```text
FullName:
C:\CurlWgetDownloadLab\benign-download.txt

Length:
334 bytes

CreationTime:
23-08-2026 07:29:10

LastWriteTime:
23-08-2026 07:29:10
```

The file contents were then examined:

```powershell
Get-Content "C:\CurlWgetDownloadLab\benign-download.txt"
```

The file contained an HTTP error response:

```text
Bad Request - Invalid Hostname
```

The complete response was an HTML HTTP 400 error page.

### Finding

The curl transfer created a file, but the expected benign text file was **not** retrieved.

Instead, the output file contained an HTTP 400 response.

This is an important distinction:

```text
curl transfer:
Successful data transfer

Expected content:
Not retrieved

Actual content:
HTTP 400 HTML error response
```

A SOC analyst should therefore verify downloaded content instead of assuming that a successful transfer percentage means the intended payload was retrieved.

---

# 8. Sysmon Investigation

The Windows Sysmon Operational log was reviewed.

The Event Viewer showed a large number of Sysmon events:

```text
Operational Number of events: 49,353
```

Filtering for Event ID 1 produced:

```text
Filtered Log:
Microsoft-Windows-Sysmon/Operational

Event ID:
1

Number of events:
15,467
```

The relevant curl process creation event occurred at approximately:

```text
23 August 2026 07:29:10
```

Important event details included:

```text
Event ID:
1

Task Category:
Process Create

Image:
C:\WINDOWS\System32\curl.exe

FileVersion:
8.13.0

Description:
The curl executable

Product:
The curl executable

Process ID:
18076

Computer:
DESKTOP-9MMM37V

User:
SYSTEM
```

### Primary Telemetry Finding

**Sysmon Event ID 1 successfully confirmed the execution of `curl.exe`.**

---

# 9. Wazuh Agent Verification

The Wazuh manager was used to verify the status of the Windows endpoint.

Command:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

Observed:

```text
Agent ID:
001

Agent Name:
DESKTOP-9MMM37V

IP address:
any

Status:
Active

Operating system:
Microsoft Windows 11 Pro

Client version:
Wazuh v4.12.0
```

This confirms that the Windows endpoint was registered with Wazuh and that the Wazuh agent was active.

---

# 10. Additional Wazuh Telemetry

A separate Wazuh event was observed from the Linux manager:

```text
rule.id:
5501

rule.description:
PAM: Login session opened.

rule.groups:
pam, syslog, authentication_success
```

This demonstrates that Wazuh was processing Linux-side telemetry.

However, this event is **not evidence of the Windows curl download** and should not be correlated with the curl activity.

---

# 11. Telemetry Coverage

The following telemetry was successfully obtained:

| Telemetry | Status |
|---|---|
| curl.exe execution | Confirmed |
| Sysmon Event ID 1 | Confirmed |
| Downloaded file creation | Confirmed through filesystem inspection |
| Downloaded file content | Confirmed |
| Wazuh agent status | Confirmed Active |

The following telemetry was **not obtained during this session**:

| Event | Status |
|---|---|
| Sysmon Event ID 3 — Network Connection | Not obtained |
| Sysmon Event ID 11 — File Create | Not obtained |
| Windows Security Event ID 4688 — Process Creation | Not obtained |
| PowerShell Event ID 4104 — Script Block Logging | Not obtained |

The correct interpretation is **"not obtained"**, not **"not generated."**

The available evidence does not establish whether these events were generated but not collected, filtered, unavailable, or never generated.

---

# 12. Investigation Assessment

The observed activity was intentionally generated as a benign laboratory exercise.

The evidence confirms:

1. `curl.exe` was executed.
2. The executable was the native Windows binary.
3. A 334-byte file was created.
4. The file contained an HTTP 400 error response.
5. Sysmon Event ID 1 recorded the curl process.
6. The Wazuh agent was active.

There is no evidence from this lab that a malicious payload was downloaded or executed.

### SOC Verdict

**Insufficient Evidence / Benign Lab Activity**

The activity should not be classified as malicious based solely on the available evidence.

In a production environment, however, unexpected `curl.exe` activity would warrant additional investigation, particularly if it were associated with an unusual parent process, external destination, executable download, or subsequent execution.

---

# 13. MITRE ATT&CK Relevance

## T1105 — Ingress Tool Transfer

Command-line utilities such as `curl.exe` can be abused to transfer tools or payloads onto a compromised endpoint.

This lab demonstrates the endpoint behavior associated with file transfer, but the simulated download did not retrieve malware.

## T1059 — Command and Scripting Interpreter

The activity was initiated through PowerShell using the command-line `curl.exe` utility.

More specific ATT&CK mapping would depend on the confirmed parent process and complete command-line telemetry.

---

