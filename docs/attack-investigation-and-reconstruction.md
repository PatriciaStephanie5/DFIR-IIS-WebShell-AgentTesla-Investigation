# Attack Investigation & Reconstruction

## Chronological Attack Reconstruction

| Phase | Relative Time / Timestamp | Artifact Source | Technical Details & Findings |
| :--- | :--- | :--- | :--- |
| **1. Reconnaissance** | `0.00s to 240.00s` (PCAP) | `capture.pcapng` (`http.request`) | Attacker (`10.0.2.4`) floods `10.0.2.15` with `4,166` packets using **Nmap Scripting Engine** (`PROPFIND` / HTTP enumeration probes). |
| **2. Share Discovery** | `240.78s to 263.31s` (PCAP) | `capture.pcapng` (`smb2.cmd == 3`) | Attacker enumerates SMB shares via Tree Connect requests to `\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`. |
| **3. Web Shell Drop** | `342.48s` (PCAP Packet `2783`) | `capture.pcapng` (`smb2.cmd == 5`) | Attacker writes **`shell.aspx`** into the web-accessible share over SMB2. |
| **4. Reverse Shell** | `401.50s` (PCAP Packet `3585`) | `capture.pcapng` & `windows.netscan` | Triggering `shell.aspx` forces IIS worker **`w3wp.exe` (PID `4332`)** to open an outbound TCP reverse shell from port `49688` to **`10.0.2.4:4443`**. |
| **5. Persistence** | `2024-09-10 06:08:23 UTC` | `memdump.mem` (`windows.pstree`) | **`w3wp.exe` (PID `4332`)** spawns **`updatenow.exe` (PID `900`)** located at `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\updatenow.exe`. |
| **6. C2 Beaconing** | Post-Execution | Static & Threat Intel Analysis | `updatenow.exe` (UPX-packed **AgentTesla**) initiates C2 communications with **`cp8nl.hyperhost.ua`**. |

---

## Forensic Deep-Dive

### Network Layer (`capture.pcapng`)
* Endpoint statistics confirmed asymmetric probe traffic from `10.0.2.4` (`4,166` transmitted packets) targeting `10.0.2.15`.
* HTTP header inspection revealed `Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org)`, confirming active NSE web enumeration.
* Filtering for SMB2 file creation events (`smb2.cmd == 5`) isolated the exact upload of `shell.aspx` into the exposed share, bridging external network access into remote code execution on IIS.

### Memory Layer (`memdump.mem`)
* Running `windows.info` established the Windows Server 2019 (Build `17763`) kernel context at base address **`0xf80079213000`**.
* Process tree inspection (`windows.pstree`) exposed a textbook parent-child anomaly: the IIS worker process `w3wp.exe` (`PID 4332`) spawned `updatenow.exe` (`PID 900`) directly out of the system-wide `StartUp` folder, ensuring execution across system reboots.
* Socket correlation (`windows.netscan`) confirmed `w3wp.exe` (`PID 4332`) owned the outbound connection to `10.0.2.4` on port `4443`.

### Malware Layer (`updatenow.exe`)
* **SHA-256 Hash:** `C25A6673A24D169DE1BB399D226C12CDC666E0FA534149FC9FA7896EE61D406F`
* Static inspection revealed **UPX** packing headers (`UPX0`/`UPX1`) used to hinder signature detection.
* Threat intelligence attribution identified the unpacked payload as **AgentTesla**, an infostealer and remote access trojan configured to beacon to **`cp8nl.hyperhost.ua`**.
