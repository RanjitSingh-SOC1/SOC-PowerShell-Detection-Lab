# 🔐 Detecting Credential Dumping (LSASS Memory Abuse) | Sysmon & Splunk

> **Detecting OS Credential Dumping (MITRE ATT&CK T1003.001) using Microsoft Sysmon telemetry and Splunk Enterprise SIEM.**

---

## 📌 Project Overview

This project demonstrates the detection, centralized log ingestion, and analysis of an **OS Credential Dumping** attack — a critical privilege escalation and lateral movement technique.

Using **Microsoft Sysinternals Procdump** on a target Windows 10 VM, I simulated an attack targeting the volatile memory of the **Local Security Authority Subsystem Service (LSASS)**. Although local Windows Defender interdiction blocked the dump file generation, the **Splunk Universal Forwarder** successfully shipped deep process telemetry to a centralized **Splunk Enterprise** indexer, enabling forensic analysis and the creation of an Incident Response Playbook.

---

## 📂 Access the Full Project Report

📄 **[Download / View the Complete Project PDF Report](./Ranjit_Singh_SOC_Project_2.pdf)**

---

## 🛠️ Technologies & Tools Used

| Component | Technology |
| :--- | :--- |
| **SIEM** | Splunk Enterprise (Centralized Indexer & Search Head) |
| **Endpoint Telemetry** | Microsoft Sysmon (SwiftOnSecurity Configuration) |
| **Log Transport** | Splunk Universal Forwarder (Port 9997 TCP) |
| **Hypervisor** | Oracle VM VirtualBox |
| **Target Host** | Windows 10 Pro (Virtual Machine) — `DESKTOP-S7EO8FC` |
| **SIEM Host** | Windows 11 Pro (Physical Host) |
| **Simulation Utility** | Microsoft Sysinternals Procdump |
| **Adversary Technique** | OS Credential Dumping: LSASS Memory (MITRE ATT&CK T1003.001) |

---

## 📐 Lab Architecture & Data Routing Pipeline

```text
+------------------------------------------+                              +-----------------------------------------+
|             WINDOWS 10 VM                |      Port 9997 (TCP)         |            WINDOWS 11 HOST              |
|        (Target Host / Forwarder)         |  ------------------------->  |          (SIEM Host / Collector)        |
|                                          |                              |                                         |
|  [Sysmon Operational Event Logs]         |                              |  [Splunk Enterprise Instance]           |
|  [Splunk Universal Forwarder Service]    |                              |  [Centralized Indexer & Search Head]    |
+------------------------------------------+                              +-----------------------------------------+
```

---

## ⚡ Attack Simulation & Endpoint Interdiction

To simulate credential harvesting, I executed an administrative command inside an elevated console on the Windows 10 target endpoint to write the memory space of `lsass.exe` to a dump file:

```cmd
C:\Users\Public\procdump.exe -accepteula -ma lsass.exe C:\Users\Public\lsass.dmp
```

**Endpoint Result:** Microsoft Defender Antivirus successfully flagged the behavior as a high-severity threat, generating a local system alert and blocking the creation of the dump file (*Access is denied*).

---

## 🔍 Forensic Analysis inside Splunk

Despite local process termination, Sysmon successfully recorded the process execution before termination. Using Splunk, I investigated the execution parameters with the following high-fidelity query:

```spl
index=* "procdump.exe" EventCode=1
```

### Extracted Forensic Indicators of Compromise (IOCs)

| Forensic Field | Discovered Value | Security Significance |
| :--- | :--- | :--- |
| **Target Endpoint** | `DESKTOP-S7EO8FC` | The victim machine targeted by the privilege escalation attempt |
| **Log Source** | `WinEventLog:Microsoft-Windows-Sysmon/Operational` (Event ID 1) | Confirms Sysmon captured process-level activity |
| **Executed Utility** | `procdump.exe` | Legitimate admin tool abused for malicious purpose (LotL) |
| **Target Process** | `lsass.exe` via `-ma` flag | Full memory dump request — classic credential harvesting |
| **Parent Process** | `cmd.exe` | Confirms manual terminal execution, not automated |
| **MD5 Hash** | `969590F449B9BB5962C6420B8AF3D7D7` | Cryptographic IOC for signature matching and blocklisting |

---

## 🛡️ Incident Response Playbook (L1 SOC Workflow)

Upon identifying this execution profile, the following operational steps are triggered:

1. **Network Containment** — Isolate host `DESKTOP-S7EO8FC` from the active network segment immediately using EDR host-isolation protocols.
2. **Credential Invalidation** — Terminate active sessions and enforce an immediate password reset for the compromised administrator account (`DESKTOP-S7EO8FC\HomeLab`).
3. **Indicator Blocklisting** — Add the MD5/SHA256 signature hashes of the `procdump.exe` execution file to the centralized EDR/AV blocklist.
4. **Attack Surface Reduction (ASR)** — Enable Windows Defender ASR Rule ID `9e6c4e1f-7d60-472f-ba1a-a39ef669e4b2` globally via Group Policy to prevent applications from obtaining unauthorized handles to LSASS.

---

## 🎓 Skills & Key Competencies Demonstrated

- **Advanced Threat Analysis:** Understanding credential harvesting techniques and corresponding memory protection architectures
- **Telemetry Correlation:** Investigating parsed JSON/XML data in a SIEM to track process creation metadata and process lineage
- **Proactive Hardening:** Mapping detection gaps directly to preventive ASR and GPO security controls
- **Incident Response:** Structuring triage, containment, eradication and recovery steps into a repeatable L1 playbook

---

## 👤 Connect with Me

**Author:** Ranjit Singh — CompTIA Security+ (SY0-701) Certified
**LinkedIn:** [linkedin.com/in/ranjit-singh-a878a93b9](https://www.linkedin.com/in/ranjit-singh-a878a93b9)
**GitHub:** [github.com/RanjitSingh-SOC1](https://github.com/RanjitSingh-SOC1)
