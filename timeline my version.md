# Timeline

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

