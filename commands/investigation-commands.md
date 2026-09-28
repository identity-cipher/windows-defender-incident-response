# Microsoft Defender Investigation Commands

This document contains the PowerShell commands used during the Microsoft Defender endpoint investigation.

The commands were used to verify scan activity, review Microsoft Defender detections, analyze threat execution status, validate the scope of the detections, perform remediation, and conduct post-remediation validation.

Personal user information has been removed or represented using environment variables where appropriate.

---

## 1. Defender Status and Scan Verification

### Review Full Scan Status

```powershell
Get-MpComputerStatus |
Select-Object FullScanStartTime, FullScanEndTime, FullScanAge
```

**Purpose:** Verify whether the Microsoft Defender full system scan completed and review its start time, completion time, and scan age.

During the investigation, this command confirmed that the full scan completed successfully.

---

## 2. Detection Analysis

### Review Defender Detection Records

```powershell
Get-MpThreatDetection
```

**Purpose:** Review Microsoft Defender detection history and associated information such as detection time, resources, remediation information, and threat status.

The `Resources` field was particularly important during the investigation because it helped identify where the detected content was located.

### Review Detailed Threat Information

```powershell
Get-MpThreat
```

**Purpose:** Review Defender threat information and properties used during execution-status analysis.

---

## 3. Threat Execution Analysis

### Group Threats by Execution and Active Status

```powershell
Get-MpThreat |
Group-Object DidThreatExecute, IsActive |
Select-Object Count, Name
```

**Purpose:** Group Defender threat records by the `DidThreatExecute` and `IsActive` properties.

The reviewed results contained 260 records grouped as:

```text
260 False, True
```

The `IsActive` value was not interpreted by itself as evidence that 260 malicious processes were actively running. The execution information was evaluated alongside Defender's resource and detection data.

### Search for Threats Reported as Executed

```powershell
Get-MpThreat |
Where-Object {$_.DidThreatExecute -eq $true}
```

**Purpose:** Specifically search the Defender threat records for identified threats reported with `DidThreatExecute = True`.

The command returned no results in the reviewed output.

---

## 4. Source and Scope Validation

### Identify Resources Outside the Kali Linux Source

```powershell
Get-MpThreatDetection |
Where-Object { ($_.Resources -join " ") -notmatch "kali-linux" } |
Select-Object InitialDetectionTime, ThreatID, ThreatStatusID,
ActionSuccess, CurrentThreatExecutionStatusID, Resources |
Format-List
```

**Purpose:** Filter Defender detection records to determine whether the reviewed data contained detection resources outside the identified Kali Linux source.

The query returned no additional resources in the reviewed output.

This result was evaluated together with the resource paths that identified offensive-security content within the Kali Linux installation ISO.

---

## 5. Remediation

After completing the source, execution, and scope analysis, the identified Kali Linux ISO was removed from the Windows endpoint.

### Rename the ISO

```powershell
Rename-Item "$env:USERPROFILE\Downloads\kali-linux-2026.2-installer-amd64.iso" "kali-delete.iso"
```

**Purpose:** Rename the identified ISO before removal.

Using `$env:USERPROFILE` prevents the documentation from exposing the personal Windows username contained in the original file path.

### Remove the ISO

```powershell
Remove-Item "$env:USERPROFILE\Downloads\kali-delete.iso" -Force
```

**Purpose:** Remove the identified Kali Linux ISO from the endpoint.

### Verify Removal

```powershell
Test-Path "$env:USERPROFILE\Downloads\kali-delete.iso"

Test-Path "$env:USERPROFILE\Downloads\kali-linux-2026.2-installer-amd64.iso"
```

**Purpose:** Verify that neither the renamed file nor the original Kali Linux ISO remained at the tested paths.

Both commands returned:

```text
False
```

This confirmed that the identified ISO was no longer present at either tested location.

---

## 6. Post-Remediation Validation

### Update Microsoft Defender Security Intelligence

```powershell
Update-MpSignature
```

**Purpose:** Update Microsoft Defender security intelligence before performing post-remediation validation.

### Start a Quick Scan

```powershell
Start-MpScan -ScanType QuickScan
```

**Purpose:** Initiate a Microsoft Defender Quick Scan following remediation.

### Verify Quick Scan Completion

```powershell
Get-MpComputerStatus |
Select-Object QuickScanStartTime, QuickScanEndTime, QuickScanAge
```

**Purpose:** Verify that the post-remediation Quick Scan completed successfully.

### Review Recent Detection Activity

```powershell
Get-MpThreatDetection |
Where-Object {$_.InitialDetectionTime -gt (Get-Date).AddHours(-2)} |
Select-Object InitialDetectionTime, ThreatID, ActionSuccess,
CurrentThreatExecutionStatusID, Resources |
Format-List
```

**Purpose:** Review Defender detection records generated during the post-remediation review period.

The query returned no results in the reviewed output.

---

## Investigation Workflow

The PowerShell analysis followed the investigation sequence:

**Scan Verification → Detection Analysis → Source Identification → Execution Analysis → Scope Validation → Remediation → Post-Remediation Validation**

The investigation focused on correlating multiple pieces of Defender evidence rather than treating detection names alone as proof of malware execution.

## Important Interpretation

The results documented here represent the Microsoft Defender data reviewed during this investigation. The absence of reported threat execution or additional post-remediation detection records was not treated as proof that the endpoint was free from every possible security threat.
