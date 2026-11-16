# Project Overview

## Investigation Scope & Objectives
TechNova Systems detected anomalous outbound network activity originating from its public-facing Microsoft IIS web server (`10.0.2.15`). A multi-artifact DFIR investigation was launched to:
1. Determine the initial intrusion vector and attacker origin from network packet captures (`capture.pcapng`).
2. Reconstruct post-exploitation execution and persistence mechanisms from volatile memory (`memdump.mem`).
3. Classify the recovered secondary payload (`updatenow.exe`) and identify external Command-and-Control (C2) infrastructure.
4. Deliver actionable remediation guidance for both TechNova Systems' executive leadership and technical engineering teams.

## Environment & Asset Summary

| Asset / Entity | Identifier / IP | Role in Incident |
| :--- | :--- | :--- |
| **Victim IIS Server** | `10.0.2.15` (Kernel Base: `0xf80079213000`) | Public-facing Windows Server hosting IIS (`w3wp.exe`) and exposed SMB shares. |
| **Threat Actor Host** | `10.0.2.4` | Originating source of Nmap scans, SMB file upload, and reverse shell listener (`TCP/4443`). |
| **External C2 Infrastructure** | `cp8nl.hyperhost.ua` | External Command-and-Control domain contacted by the implanted AgentTesla payload. |

## Forensic Toolkit
* **Network Forensics:** Wireshark (Endpoint statistics, HTTP stream reconstruction, SMB2 command filtering)
* **Memory Forensics:** Volatility 3 (`windows.info`, `windows.pstree`, `windows.netscan`)
* **Malware Triage:** PowerShell (`Get-FileHash`), Static string inspection, VirusTotal & MalwareBazaar
