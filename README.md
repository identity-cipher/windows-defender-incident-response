# Microsoft Defender Endpoint Investigation

> Endpoint security investigation using Microsoft Defender and PowerShell for detection analysis, remediation, and post-incident validation.

## Overview

This project documents a Windows endpoint security investigation performed after Microsoft Defender identified multiple security-tool and potentially malicious signatures during a full system scan.

The goal of the investigation was to determine the source of the detections, assess whether Defender reported any of the identified threats as having executed, identify whether detection resources existed outside the suspected source, and complete remediation and post-remediation validation.

Using Microsoft Defender, PowerShell, and Defender event data, I traced the reviewed detections to offensive-security tools contained within a Kali Linux installation ISO that had previously been downloaded for use in a virtual lab environment.

The investigation was completed across multiple sessions in a personal lab environment. Timestamps shown in the evidence reflect the actual investigation and remediation timeline.

## Investigation Objectives

The investigation focused on answering the following questions:

- What did Microsoft Defender detect?
- What was the source of the detected content?
- Did Defender report any identified threats as having executed?
- Were detection resources identified outside the suspected source?
- Could the identified source be safely removed?
- Did additional detections appear during post-remediation validation?

## Environment & Tools

| Technology | Purpose |
|---|---|
| Windows | Endpoint under investigation |
| Microsoft Defender Antivirus | Endpoint scanning and threat detection |
| PowerShell | Detection analysis, filtering, remediation, and validation |
| Microsoft Defender event data | Review of scan and detection activity |
| Kali Linux ISO | Source investigated during detection analysis |
| Oracle VirtualBox | Virtual lab environment associated with the Kali ISO |

## Investigation

### Phase 1 — Detection & Initial Analysis
<img width="768" height="341" alt="01-full-scan-completed" src="https://github.com/user-attachments/assets/1e38f303-a19b-4234-81eb-bbcd84999d95" />

*Figure 1. PowerShell verification of the completed Microsoft Defender full system scan.*

