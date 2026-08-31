# Investigation Notes 
---

## 1. Environment Validation

The investigation was performed on:

- **Hostname:** `DESKTOP-9MMM37V`
- **User:** `DESKTOP-9MMM37V\dell`
- **PowerShell:** `7.6.5`
- **Lab Directory:** `C:\AppInitDLLLab`

The initial environment checks confirmed the endpoint identity and PowerShell version before beginning the investigation.

---

## 2. Lab Directory Preparation

The investigation workspace was created under:

`C:\AppInitDLLLab`

The following directories were prepared:

```text
C:\AppInitDLLLab\
├── Payload\
├── Output\
└── Evidence\
```

The directory structure separates payload preparation, expected execution output, and collected evidence.

---

## 3. Benign DLL Preparation

A controlled source file was created:

`C:\AppInitDLLLab\Payload\AppInitTest.cpp`

The DLL was intended to perform a harmless action when loaded by creating a marker file:

`C:\AppInitDLLLab\Output\appinit-loaded.txt`

The payload was designed only for controlled lab validation and did not include networking, credential access, additional persistence, or other malicious functionality.

---

## 4. Source File Verification

The source file metadata was collected during the investigation.

```text
FullName:
C:\AppInitDLLLab\Payload\AppInitTest.cpp

Length:
915 bytes

CreationTime:
31-08-2026 08:54:13

LastWriteTime:
31-08-2026 08:54:13
```

SHA256:

`714EFF2162D100CCD835E817B25F611A29F89D2C5BB04C26581153C171DD1BCC`

> This hash belongs to the `.cpp` source file and not to the compiled DLL.

---

## 5. DLL Verification

The expected compiled DLL was:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

The captured verification returned:

```text
Get-Item: Cannot find path
'C:\AppInitDLLLab\Payload\AppInitTest.dll'
because it does not exist.
```

This means the compiled DLL could not be confirmed during the captured investigation.

As a result:

- DLL metadata could not be verified.
- DLL SHA256 could not be collected.
- The DLL could not be directly correlated with subsequent telemetry.
- Successful DLL execution could not be established.

---

## 6. AppInit Registry Analysis

The following registry location was examined:

`HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`

The captured state was:

```text
AppInit_DLLs     :
LoadAppInit_DLLs : 0
```

The observed `LoadAppInit_DLLs` value of `0` does not demonstrate an enabled AppInit DLL-loading configuration at the time of collection.

This became an important limitation when assessing whether the persistence mechanism was actually active.

---

## 7. Controlled Process Execution

A normal Windows process was launched:

```powershell
Start-Process notepad.exe
```

Approximate execution time:

`31 August 2026 09:03:42`

The purpose was to generate controlled process activity that could be examined using Sysmon.

---

## 8. Sysmon Event ID 1 Analysis

Sysmon Event ID 1 was reviewed for process creation activity.

The captured output showed multiple process creation events approximately between:

`09:03:35 – 09:05:08`

This confirms that process activity occurred during the investigation.

However, process creation alone does not demonstrate that `AppInitTest.dll` was loaded into the process.

### Assessment

- **Process creation:** Confirmed
- **AppInit DLL execution:** Not confirmed

---

## 9. Sysmon Event ID 11 Analysis

Sysmon Event ID 11 was reviewed for file creation activity.

A detailed event showed:

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

This demonstrates PowerShell-related file creation under the SYSTEM context.

The event is useful as endpoint telemetry but does not directly establish AppInit DLL execution.

---

## 10. Sysmon Event ID 7 Analysis

Sysmon Event ID 7 would provide stronger evidence if it showed an image-load event for:

`C:\AppInitDLLLab\Payload\AppInitTest.dll`

No confirmed Event ID 7 record for the expected DLL was demonstrated in the captured evidence.

Possible explanations include:

- Event ID 7 was not enabled.
- The DLL was not present.
- The DLL was not loaded.
- AppInit configuration was incomplete.
- Available telemetry was insufficient.

The absence of Event ID 7 is therefore treated as an evidence limitation rather than definitive proof that DLL loading did not occur.

---

## 11. Marker File Verification

The benign DLL was expected to create:

`C:\AppInitDLLLab\Output\appinit-loaded.txt`

The captured evidence does not demonstrate creation of this file.

Therefore, the expected execution marker cannot be used to confirm DLL execution.

---

