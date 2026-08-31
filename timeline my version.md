# Timeline 

## Investigation Timeline

| Time | Source | Event | Significance |
|---|---|---|---|
| 08:42:53 | PowerShell | Hostname, user, date, and PowerShell version collected | Environment baseline established |
| 08:43 | PowerShell | `C:\AppInitDLLLab` created | Investigation workspace initialized |
| 08:43 | PowerShell | `Payload`, `Output`, and `Evidence` directories created | Lab structure prepared |
| 08:54:13 | File | `AppInitTest.cpp` created | Benign DLL source established |
| 08:54:13 | File | `AppInitTest.cpp` last modified | Source timestamp recorded |
| Pre-09:03 | Registry | AppInit registry configuration queried | Persistence configuration examined |
| Pre-09:03 | Registry | `LoadAppInit_DLLs : 0` observed | AppInit loading not enabled in captured state |
| Pre-09:03 | PowerShell | `AppInitTest.dll` verification attempted | Expected DLL not found |
| 09:03:42 | PowerShell | `Start-Process notepad.exe` executed | Controlled process activity generated |
| 09:03:35–09:05:08 | Sysmon Event ID 1 | Process creation activity observed | Process telemetry generated |
| 09:03:20–09:06:00 | Sysmon Event ID 11 | File creation activity observed | File telemetry generated |
| 03:37:54 UTC | Sysmon Event ID 11 | PowerShell created temporary `.ps1` file | SYSTEM-level PowerShell file activity observed |

---

