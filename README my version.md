# windows-dfir-lab67-appinit-dll-persistence-investigation

## Overview

AppInit DLLs are a Windows persistence mechanism where DLLs can be configured to load into processes that use User32.dll. The configuration is controlled through specific Windows Registry values, primarily under:

HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows

The important values include:

AppInit_DLLs
LoadAppInit_DLLs

From a DFIR/SOC perspective, an unexpected DLL listed in AppInit_DLLs can be significant because it may cause a DLL to be loaded automatically into multiple user-mode processes.

The investigation should therefore focus on the relationship:

Registry Configuration
        ↓
AppInit_DLLs
        ↓
Referenced DLL
        ↓
Process Loading
        ↓
Sysmon / Wazuh Telemetry
        ↓
Persistence Assessment

However, the presence of an AppInit configuration alone does not prove malicious activity. AppInit has legitimate historical uses, and modern Windows versions have additional protections and limitations. The analyst needs to establish whether the configured DLL is expected, where it resides, who created or modified it, and whether process telemetry supports its loading.

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

- Understand how AppInit DLL persistence can be investigated from a Windows DFIR/SOC perspective.
- Examine the relevant Windows registry configuration and determine its significance.
- Prepare and verify a controlled benign DLL and collect relevant file metadata.
- Generate controlled process activity for endpoint telemetry analysis.
- Analyze Sysmon Event ID 1 for process creation activity.
- Examine Sysmon Event ID 7 for potential DLL image-load evidence.
- Analyze Sysmon Event ID 11 for supporting file-creation activity.
- Verify whether the expected benign execution artifact was created.
- Correlate registry, file, process, and DLL-loading evidence into an execution chain.
- Identify missing or insufficient telemetry and document investigation limitations.
- Produce an evidence-based conclusion without overstating an unconfirmed persistence mechanism.

---

## Investigation Scenario

A Windows endpoint is suspected of using AppInit DLL persistence. The analyst investigates the registry configuration, benign DLL, process activity, and Sysmon telemetry to determine whether the suspected persistence mechanism was actually active and executed.

The investigation focuses on:

- Examining the AppInit registry configuration.
- Verifying the expected DLL and its metadata.
- Reviewing Sysmon process, file, and DLL-load activity.
- Checking for the expected benign execution marker.
- Correlating the collected artifacts.
- Determining whether the evidence confirms or only suggests AppInit persistence.

Any missing or conflicting evidence must be documented rather than treated as proof of successful persistence.

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
