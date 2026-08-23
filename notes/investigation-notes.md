# Investigation Notes — Lab 59

## Curl / Wget Download Investigation

---

## 1. Case Information

| Field | Value |
|---|---|
| Lab | Lab 59 |
| Investigation | Curl / Wget Download Investigation |
| Date | 23 August 2026 |
| Host | DESKTOP-9MMM37V |
| User | desktop-9mmm37v\dell |
| Operating System | Windows 11 Pro |
| PowerShell | 7.6.5 |
| curl Version | 8.13.0 |
| curl Path | `C:\Windows\System32\curl.exe` |
| Wazuh Agent | 001 |
| Wazuh Version | 4.12.0 |
| Wazuh Status | Active |
| Primary Confirmed Event | Sysmon Event ID 1 |

---

## 2. Investigation Objective

The objective was to simulate a controlled command-line download using the native Windows `curl.exe` utility and determine what endpoint telemetry was available for the activity.

The investigation focused on:

- Endpoint identification.
- Filesystem baselining.
- curl binary verification.
- Download execution.
- File verification.
- Sysmon process telemetry.
- Wazuh agent status.
- Identification of telemetry gaps.

---

## 3. Initial Endpoint Identification

The following commands were executed:

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

This established the identity of the system and user involved in the laboratory activity.

---

## 4. Investigation Directory

The investigation workspace was created using:

```powershell
New-Item -Path "C:\CurlWgetDownloadLab" -ItemType Directory -Force
```

The directory was then verified:

```powershell
Get-Item "C:\CurlWgetDownloadLab"
```

The directory was used as the destination for the curl download.

---

## 5. Baseline Collection

Before the download, the directory contents were recorded:

```powershell
Get-ChildItem "C:\CurlWgetDownloadLab" -Force |
Select-Object Name, Length, CreationTime, LastWriteTime |
Out-File "C:\CurlWgetDownloadLab\download-baseline.txt"
```

The purpose was to establish the initial filesystem state.

This provides a simple before-and-after comparison for the investigation.

---

## 6. Controlled Download Source

A separate directory was created:

```powershell
New-Item -Path "C:\CurlDownloadServer" -ItemType Directory -Force
```

The intended benign test file was created:

```powershell
"This is a benign file used for Curl download investigation." |
Out-File "C:\CurlDownloadServer\benign-download.txt"
```

The expected content was:

```text
This is a benign file used for Curl download investigation.
```

---

## 7. Network Configuration

The endpoint network configuration was checked:

```powershell
ipconfig
```

The relevant VMware adapter showed:

```text
Adapter:
VMware Network Adapter VMnet1

IPv4:
192.168.174.1

Subnet Mask:
255.255.255.0
```

Other listed Ethernet and wireless adapters were disconnected.

---

## 8. curl Verification

The installed curl executable was identified using:

```powershell
Get-Command curl.exe
```

Result:

```text
C:\Windows\System32\curl.exe
```

The version was confirmed:

```powershell
curl.exe --version
```

Result:

```text
curl 8.13.0
libcurl/8.13.0
Schannel
```

The binary was therefore confirmed as the Windows system curl executable.

---

## 9. Download Command

The controlled download was executed at approximately 07:28:10.

Command:

```powershell
curl.exe http://127.0.0 -o C:\CurlWgetDownloadLab\benign-download.txt
```

Curl reported:

```text
100   334
```

The output file was then inspected.

---

## 10. Download Artifact

The file was identified using:

```powershell
Get-Item "C:\CurlWgetDownloadLab\benign-download.txt"
```

Observed:

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

The file content was inspected:

```powershell
Get-Content "C:\CurlWgetDownloadLab\benign-download.txt"
```

The result was an HTML HTTP error response:

```text
Bad Request - Invalid Hostname
```

---

## 11. File Analysis

The intended file content was:

```text
This is a benign file used for Curl download investigation.
```

The actual downloaded content was:

```text
HTTP 400 Bad Request
Invalid Hostname
```

Therefore, the download operation should be interpreted as:

```text
curl execution:
Confirmed

Data transfer:
Confirmed

File creation:
Confirmed

Expected benign file retrieval:
Not confirmed

Actual content:
HTTP 400 error page
```

This is a useful investigation finding because it demonstrates why analysts should inspect the resulting file rather than relying only on the transfer status.

---

## 12. Sysmon Evidence

The Sysmon Operational log was filtered for Event ID 1.

The relevant event showed:

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

Timestamp:

```text
23 August 2026 07:29:10
```

### Evidence Assessment

This is the strongest endpoint telemetry collected during the investigation.

It directly confirms that `curl.exe` was executed.

---

## 13. Wazuh Agent Verification

The Wazuh manager was queried using:

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

The Windows endpoint was therefore actively connected to Wazuh.

---

## 14. Telemetry Obtained

The investigation successfully obtained:

| Evidence | Result |
|---|---|
| Endpoint identity | Confirmed |
| User identity | Confirmed |
| curl executable | Confirmed |
| curl version | Confirmed |
| Download command | Confirmed |
| Download output file | Confirmed |
| File size | 334 bytes |
| File content | HTTP 400 response |
| Sysmon Event ID 1 | Confirmed |
| Wazuh agent status | Active |

---

## 15. Telemetry Not Obtained

The following events were not successfully obtained:

| Event | Purpose | Result |
|---|---|---|
| Sysmon Event ID 3 | Network Connection | Not obtained |
| Sysmon Event ID 11 | File Create | Not obtained |
| Security Event ID 4688 | Process Creation | Not obtained |
| PowerShell Event ID 4104 | Script Block Logging | Not obtained |

These should be recorded as telemetry gaps.

They should not be described as events that definitely did not occur.

---

## 16. Investigation Limitations

The investigation cannot currently establish:

- The complete curl command line from Sysmon telemetry.
- Network connection telemetry associated with the curl process.
- The destination IP/domain from endpoint telemetry.
- Sysmon File Create telemetry.
- Windows Security Event 4688.
- PowerShell Script Block Event 4104.
- Whether the downloaded file was subsequently executed.
- Whether any persistence or follow-on activity occurred.

---

## 17. SOC Analysis

The execution of `curl.exe` alone is not sufficient evidence of malicious activity.

Curl is a legitimate command-line utility and can be used for:

- Administrative downloads.
- Testing.
- Software installation.
- Automation.
- Troubleshooting.
- File retrieval.

The same utility can also be abused by attackers for payload delivery.

Therefore, a production SOC alert should evaluate the surrounding context.

Useful correlation points include:

```text
curl.exe
    +
Parent process
    +
Command line
    +
Network destination
    +
Downloaded file
    +
File type
    +
File hash
    +
Subsequent execution
```

---

## 18. Investigation Verdict

### Verdict: Insufficient Evidence

The activity was intentionally generated as a benign laboratory exercise.

The available evidence does not indicate malicious payload delivery or execution.

The correct SOC classification for this lab is:

```text
Benign Laboratory Activity
```

For an unexpected production alert, the appropriate interim classification would be:

```text
Insufficient Evidence — Continue Investigation
```

---

## 19. Key Analyst Takeaway

The central lesson from this investigation is:

> A process creation event tells the analyst that a program ran, but not necessarily why it ran or what happened afterward.

A stronger investigation requires correlation across:

```text
Process
    ↓
Command Line
    ↓
Network
    ↓
File Creation
    ↓
File Analysis
    ↓
Execution
    ↓
Persistence / Follow-on Activity
```

Only Sysmon Event ID 1 was successfully obtained in this lab, so the investigation should be considered a partial telemetry exercise rather than a complete download investigation.
