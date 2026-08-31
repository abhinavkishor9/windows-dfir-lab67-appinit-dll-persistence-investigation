# Windows DFIR Lab 67 – AppInit DLL Persistence Investigation

## Description

A Windows DFIR/SOC lab investigating suspected AppInit DLL persistence using registry analysis, a benign DLL, Sysmon telemetry, and evidence correlation.

> **Investigation status:** AppInit DLL loading was **not conclusively demonstrated** in the captured evidence.

---

## Overview

This lab investigates the Windows **AppInit DLL persistence mechanism** from a SOC/DFIR perspective.

A benign DLL is used to create a harmless marker file when loaded. The investigation examines the AppInit registry configuration and correlates it with process creation, file activity, DLL-load telemetry, and execution artifacts.

The lab focuses on evidence-based conclusions rather than assuming persistence was successful simply because the relevant registry location was investigated.

---

## Lab Environment

- **OS:** Windows
- **Hostname:** `DESKTOP-9MMM37V`
- **User:** `DESKTOP-9MMM37V\dell`
- **PowerShell:** `7.6.5`
- **Telemetry:** Microsoft Sysmon
- **Relevant Sysmon Events:** 1, 7, 11
- **Lab Directory:** `C:\AppInitDLLLab`

---

## Lab Objectives

- Understand AppInit DLL persistence.
- Prepare a controlled benign DLL.
- Examine the AppInit registry configuration.
- Collect DLL metadata and SHA256 hashes.
- Generate controlled process activity.
- Investigate Sysmon process and file events.
- Look for DLL image-load telemetry.
- Correlate registry, file, process, and execution evidence.
- Document investigation gaps and limitations.
- Avoid treating incomplete evidence as confirmed persistence.

---

## Investigation Scenario

A Windows endpoint is being investigated for possible persistence through the **AppInit DLL mechanism**.

The analyst prepares a benign DLL, examines the relevant registry configuration, launches a normal Windows process, and reviews Sysmon telemetry to determine whether the DLL was actually loaded.

The investigation must distinguish between **persistence configuration** and **confirmed persistence execution**.

---

## Lab Structure

```text
C:\AppInitDLLLab\
├── Payload\
│   ├── AppInitTest.cpp
│   └── AppInitTest.dll
├── Output\
│   └── appinit-loaded.txt
└── Evidence\
    ├── dll-baseline.txt
    ├── dll-metadata.txt
    ├── dll-hash.txt
    ├── appinit-registry.txt
    └── appinit-after-change.txt
```

---

## Investigation Workflow

```text
Environment Validation
        ↓
Lab Directory Preparation
        ↓
Benign DLL Preparation
        ↓
DLL Metadata & Hash Collection
        ↓
AppInit Registry Examination
        ↓
Controlled Process Execution
        ↓
Sysmon Event ID 1 Analysis
        ↓
Sysmon Event ID 11 Analysis
        ↓
Sysmon Event ID 7 Analysis
        ↓
Marker File Verification
        ↓
Evidence Correlation
        ↓
Final Assessment
```

---

# Evidence Collection

## Source File

The benign DLL source was created at:

`C:\AppInitDLLLab\Payload\AppInitTest.cpp`

Observed metadata:

```text
Length:
915 bytes

CreationTime:
31-08-2026 08:54:13

LastWriteTime:
31-08-2026 08:54:13
```

SHA256:

`714EFF2162D100CCD835E817B25F611A29F89D2C5BB04C26581153C171DD1BCC`

> The hash above belongs to the `.cpp` source file, not the compiled DLL.

---

## DLL Verification

The expected DLL was:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

However, the captured evidence showed that the file was not present.

```text
Get-Item: Cannot find path
'C:\AppInitDLLLab\Payload\AppInitTest.dll'
because it does not exist.
```

Therefore:

- The compiled DLL could not be verified.
- A DLL SHA256 could not be collected.
- DLL execution could not be conclusively established.

---

# AppInit Registry Analysis

The relevant registry location was examined:

`HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`

Captured state:

```text
AppInit_DLLs     :
LoadAppInit_DLLs : 0
```

### Interpretation

The captured registry state does **not** demonstrate an enabled AppInit DLL-loading configuration.

This is an important limitation and should remain documented in the investigation.

---

# Process Activity

A normal Windows process was launched for controlled testing:

```powershell
Start-Process notepad.exe
```

Approximate execution time:

`31 August 2026 09:03:42`

This generated process activity that could be examined through Sysmon.

---

# Sysmon Event ID 1 – Process Creation

Sysmon Event ID 1 showed multiple process creation events during the investigation period.

Observed activity occurred approximately between:

`09:03:35 – 09:05:08`

### Assessment

Process creation was confirmed.

However, the available evidence does not establish that the AppInit DLL was loaded into any of these processes.

- **Process creation:** Confirmed
- **AppInit DLL execution:** Not confirmed

---

# Sysmon Event ID 11 – File Creation

Sysmon Event ID 11 showed multiple file creation events during the test.

One detailed event contained:

```text
Event ID:
11

Computer:
DESKTOP-9MMM37V

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

### Assessment

This confirms PowerShell-related file activity under the SYSTEM context.

The event should **not** be treated as direct evidence of AppInit DLL execution.

---

# Sysmon Event ID 7 – Image Load

Sysmon Event ID 7 would provide stronger evidence if it showed:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

However, the captured evidence does not demonstrate such an Event ID 7 record.

### Assessment

**DLL Image Load Evidence:** Not demonstrated

Possible reasons include:

- Event ID 7 was not enabled.
- The DLL was not present.
- The DLL was not loaded.
- Required AppInit configuration was incomplete.
- Available telemetry was insufficient.

The absence of Event ID 7 should therefore be documented as a **telemetry limitation**, not automatically interpreted as proof that loading did not occur.

---

# Marker File

The benign DLL was designed to create:

`C:\AppInitDLLLab\Output\appinit-loaded.txt`

The captured evidence does not demonstrate creation of this file.

Therefore:

**Benign execution marker:** Not demonstrated

---

# Evidence Correlation

The expected investigation chain was:

```text
AppInit Registry Configuration
        ↓
Configured DLL
        ↓
Process Creation
        ↓
DLL Image Load
        ↓
Benign Marker File
```

Current evidence:

| Evidence | Status |
|---|---|
| Lab environment | Confirmed |
| Lab directories | Confirmed |
| Benign source file | Confirmed |
| Source metadata | Confirmed |
| Source SHA256 | Confirmed |
| Compiled DLL | Not confirmed |
| AppInit enabled | Not confirmed |
| Process creation | Confirmed |
| File creation | Confirmed |
| DLL image load | Not demonstrated |
| Marker file | Not demonstrated |
| Successful AppInit execution | **Not confirmed** |

---

# Investigation Findings

- The AppInit investigation environment was successfully prepared.
- A benign DLL source file was created and hashed.
- The expected compiled DLL was not present during verification.
- The captured registry state showed `LoadAppInit_DLLs : 0`.
- Sysmon Event ID 1 confirmed process creation activity.
- Sysmon Event ID 11 confirmed file creation activity.
- No confirmed Sysmon Event ID 7 DLL-load evidence was captured.
- The expected benign marker file was not demonstrated.
- The available evidence does not prove successful AppInit DLL persistence.

---

# Troubleshooting Notes

## Missing DLL

The expected file:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

was not found.

If continuing the lab, compile and verify the DLL before performing persistence validation.

```powershell
Set-Location "C:\AppInitDLLLab\Payload"

cl.exe /LD AppInitTest.cpp /Fe:AppInitTest.dll

Get-Item "C:\AppInitDLLLab\Payload\AppInitTest.dll"

Get-FileHash "C:\AppInitDLLLab\Payload\AppInitTest.dll" -Algorithm SHA256
```

---

## Incorrect DLL Filename

An attempted verification used:

`test-dll.dll`

while the expected filename was:

`AppInitTest.dll`

Use one consistent filename throughout the investigation.

---

## PowerShell Continuation Error

An extra `>>` prompt was incorrectly treated as part of a command.

Correct syntax:

```powershell
Get-Item "C:\AppInitDLLLab\Payload\AppInitTest.cpp" |
Select-Object FullName, Length, CreationTime, LastWriteTime
```

---

## Sysmon Event ID 7

If Image Load monitoring is enabled, investigate Event ID 7 around the process execution time and search for:

`AppInitTest.dll`

If Event ID 7 is unavailable, document the telemetry limitation.

---

# Timeline

| Time | Source | Event | Significance |
|---|---|---|---|
| 08:42:53 | PowerShell | Host/user/date/version collected | Environment baseline |
| 08:43 | PowerShell | Lab directories created | Workspace prepared |
| 08:54:13 | File | `AppInitTest.cpp` created | Benign DLL source prepared |
| Pre-09:03 | Registry | AppInit configuration queried | Persistence state examined |
| Pre-09:03 | Registry | `LoadAppInit_DLLs : 0` | AppInit loading not enabled in captured state |
| Pre-09:03 | PowerShell | DLL verification attempted | DLL not found |
| 09:03:42 | PowerShell | `Start-Process notepad.exe` | Controlled process activity |
| 09:03:35–09:05:08 | Sysmon EID 1 | Process creation activity | Process telemetry generated |
| 09:03:20–09:06:00 | Sysmon EID 11 | File creation activity | File telemetry generated |
| 03:37:54 UTC | Sysmon EID 11 | PowerShell created temporary `.ps1` file | SYSTEM-level PowerShell file activity |

---

# Final Assessment

The investigation demonstrates the process of examining a suspected AppInit DLL persistence mechanism using registry, file, process, and Sysmon evidence.

The available evidence confirms **lab preparation and endpoint activity**, but it does not establish the complete persistence execution chain.

> **AppInit DLL persistence was investigated, but successful DLL loading and persistence were not conclusively demonstrated from the captured evidence.**

This distinction is important in SOC/DFIR investigations: **configuration evidence alone should not be reported as confirmed persistence without supporting execution telemetry.**

---

# MITRE ATT&CK Relevance

This lab is relevant to Windows persistence and execution investigation, particularly:

- **T1546.010 – Event Triggered Execution: AppInit DLLs**
- Registry artifact analysis
- Process execution analysis
- DLL/image-load monitoring
- File activity analysis
- Evidence correlation
- Timeline reconstruction
- Detection engineering

---

# Key Takeaway

```text
Registry Artifact
      +
Process Activity
      +
DLL Load Evidence
      +
Execution Artifact
      =
Strong Persistence Evidence
```

If the execution and DLL-load evidence are missing, the finding should remain **unconfirmed/inconclusive** rather than being overstated as successful persistence.
