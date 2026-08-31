# Troubleshooting Notes – AppInit DLL Persistence

## Purpose

This document records issues encountered during the AppInit DLL persistence investigation and explains how each issue affected the investigation.

---

## 1. Compiled DLL Not Found

### Issue

The expected DLL:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

was not present when verification was attempted.

Observed error:

```text
Get-Item: Cannot find path
'C:\AppInitDLLLab\Payload\AppInitTest.dll'
because it does not exist.
```

### Impact

The investigation could not verify:

- DLL metadata
- DLL SHA256
- DLL integrity
- DLL presence during testing
- Direct correlation between the DLL and Sysmon telemetry

### Recommended Resolution

Compile the source and verify the resulting DLL before continuing the persistence validation.

```powershell
Set-Location "C:\AppInitDLLLab\Payload"

cl.exe /LD AppInitTest.cpp /Fe:AppInitTest.dll

Get-Item "C:\AppInitDLLLab\Payload\AppInitTest.dll"

Get-FileHash "C:\AppInitDLLLab\Payload\AppInitTest.dll" -Algorithm SHA256
```

---

## 2. Incorrect DLL Filename

### Issue

An attempted verification referenced:

`test-dll.dll`

while the expected DLL name was:

`AppInitTest.dll`

### Impact

The incorrect filename produced a file-not-found result and could have caused confusion during evidence collection.

### Resolution

Use the same DLL filename consistently throughout the lab:

`AppInitTest.dll`

---

## 3. AppInit Registry State

### Issue

The captured registry state showed:

```text
AppInit_DLLs     :
LoadAppInit_DLLs : 0
```

### Impact

The captured evidence does not demonstrate an enabled AppInit DLL-loading configuration.

This weakens the persistence hypothesis and must be considered when interpreting subsequent process activity.

### Resolution

If the lab is continued, document any later registry changes separately and collect the registry state again after the change.

Do not overwrite or reinterpret the original captured state.

---

## 4. PowerShell Continuation Prompt

### Issue

An extra `>>` prompt was incorrectly included when entering a PowerShell command.

### Impact

PowerShell interpreted the continuation input incorrectly and generated an error.

### Resolution

Use the command without the extra continuation characters:

```powershell
Get-Item "C:\AppInitDLLLab\Payload\AppInitTest.cpp" |
Select-Object FullName, Length, CreationTime, LastWriteTime
```

---

## 5. Missing Sysmon Event ID 7 Evidence

### Issue

The investigation did not demonstrate a Sysmon Event ID 7 record showing:

`AppInitTest.dll`

### Impact

Event ID 1 confirms process creation, but it does not prove that the DLL was loaded.

Without image-load evidence, the execution chain remains incomplete.

### Possible Causes

- Sysmon Event ID 7 may not be enabled.
- The DLL may not have existed at test time.
- The DLL may not have been loaded.
- AppInit configuration may have been incomplete.
- The available telemetry may not have captured the relevant event.

### Resolution

If Event ID 7 is enabled, investigate image-load events around the controlled process execution time and search for:

`AppInitTest.dll`

---

## 6. Marker File Not Created

### Issue

The expected benign marker:

`C:\AppInitDLLLab\Output\appinit-loaded.txt`

was not demonstrated in the captured evidence.

### Impact

The marker cannot be used as evidence that the DLL executed.

### Resolution

If the lab is repeated, verify the marker location before and after controlled execution and record its creation timestamp.

---

## 7. Sysmon Event ID 11 Misinterpretation

### Issue

A Sysmon Event ID 11 event showed PowerShell creating:

`C:\Windows\SystemTemp\__PSScriptPolicyTest_2ms4hdxd.u33.ps1`

under:

`NT AUTHORITY\SYSTEM`

### Impact

This event could be incorrectly attributed to the AppInit investigation.

### Resolution

Treat the event as separate PowerShell file activity unless additional evidence establishes a direct relationship with the AppInit DLL.

---

## 8. Investigation Lesson

A persistence investigation should distinguish between:

```text
Persistence Configuration
```

and:

```text
Confirmed Persistence Execution
```

Registry configuration, process creation, file creation, DLL loading, and execution artifacts should be correlated before declaring successful persistence.

---

## Final Troubleshooting Assessment

The main investigation limitations were:

- Compiled DLL not available during verification.
- `LoadAppInit_DLLs` captured as `0`.
- No demonstrated Event ID 7 DLL-load evidence.
- Expected marker file not demonstrated.

These limitations should be documented rather than hidden or interpreted as successful persistence.
