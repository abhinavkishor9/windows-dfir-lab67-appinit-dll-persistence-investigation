# Timeline – AppInit DLL Persistence Investigation

## Timeline Overview

This timeline summarizes the captured activity during the AppInit DLL persistence investigation.

> Times are shown as captured. Sysmon records may use UTC where explicitly indicated, while PowerShell timestamps reflect the local system time.

---

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

## Key Timeline Events

### 08:42:53 – Environment Baseline

The endpoint environment was identified:

- Hostname: `DESKTOP-9MMM37V`
- User: `DESKTOP-9MMM37V\dell`
- PowerShell: `7.6.5`

This established the system context for the investigation.

---

### 08:43 – Lab Workspace Created

The investigation workspace was created under:

`C:\AppInitDLLLab`

The `Payload`, `Output`, and `Evidence` directories were prepared for the controlled investigation.

---

### 08:54:13 – Benign Source Created

The source file:

`C:\AppInitDLLLab\Payload\AppInitTest.cpp`

was created.

Observed SHA256:

`714EFF2162D100CCD835E817B25F611A29F89D2C5BB04C26581153C171DD1BCC`

---

### Pre-09:03 – AppInit Registry Examination

The relevant registry location was queried:

`HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`

Captured values:

```text
AppInit_DLLs     :
LoadAppInit_DLLs : 0
```

This showed that the captured registry state did not demonstrate enabled AppInit DLL loading.

---

### Pre-09:03 – DLL Verification

The expected DLL:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

was checked but was not found.

This prevented direct verification of the compiled DLL and its hash.

---

### 09:03:42 – Controlled Process Execution

The following command was used:

```powershell
Start-Process notepad.exe
```

This generated controlled process activity for Sysmon analysis.

---

### 09:03:35–09:05:08 – Sysmon Event ID 1

Sysmon Event ID 1 recorded multiple process creation events.

The events confirm process activity but do not independently prove AppInit DLL loading.

---

### 09:03:20–09:06:00 – Sysmon Event ID 11

Sysmon Event ID 11 recorded multiple file creation events.

These events demonstrate file activity during the investigation but do not independently confirm AppInit DLL execution.

---

### 03:37:54 UTC – PowerShell File Activity

A detailed Sysmon Event ID 11 event showed:

```text
User:
NT AUTHORITY\SYSTEM

Image:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

Process ID:
4496

Target Filename:
C:\Windows\SystemTemp\__PSScriptPolicyTest_2ms4hdxd.u33.ps1

UTC Time:
2026-08-31 03:37:54.413
```

This represents PowerShell-related file activity and is not considered direct evidence of AppInit DLL execution.

---

# Evidence Status Over Time

| Evidence Stage | Result |
|---|---|
| Environment preparation | Confirmed |
| Lab directory creation | Confirmed |
| Benign source creation | Confirmed |
| Source hashing | Confirmed |
| Compiled DLL verification | Not confirmed |
| AppInit registry configuration | Not confirmed |
| Controlled process execution | Confirmed |
| Process telemetry | Confirmed |
| File telemetry | Confirmed |
| DLL image-load telemetry | Not demonstrated |
| Benign execution marker | Not demonstrated |
| Successful AppInit persistence | **Not confirmed** |

---

# Timeline Assessment

The timeline shows a progression from environment preparation and benign payload creation to registry examination, controlled process execution, and Sysmon telemetry collection.

However, the final stages required to confirm AppInit DLL execution were not demonstrated.

The evidence therefore supports an **inconclusive AppInit persistence investigation**, rather than a confirmed persistence finding.
