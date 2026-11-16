# Technical Incident Summary - TechNova IT & Security Engineering

**Incident Title:** IIS Web Shell Intrusion (`shell.aspx`) & AgentTesla Persistence (`updatenow.exe`)  
**Affected Host:** `10.0.2.15` (Windows Server 2019 Build `17763`, Kernel Base `0xf80079213000`)  
**Verdict:** **True Positive**  

---

## Technical Narrative
1. **Reconnaissance & Initial Access:** Source IP `10.0.2.4` conducted Nmap NSE HTTP enumeration against `10.0.2.15`, enumerated SMB shares (`\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`), and uploaded `shell.aspx` over SMB2 at packet `2783`.
2. **Execution & Reverse Shell:** At packet `3585`, `shell.aspx` triggered an outbound TCP reverse shell from IIS worker process **`w3wp.exe` (`PID 4332`)** (`10.0.2.15:49688`) to **`10.0.2.4:4443`**.
3. **Persistence & C2:** `w3wp.exe` (`PID 4332`) spawned **`updatenow.exe` (`PID 900`)** inside `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\`. Static and threat-intel analysis confirmed `updatenow.exe` is a **UPX-packed AgentTesla** payload configured to communicate with **`cp8nl.hyperhost.ua`**.

---

## Indicators of Compromise (IoCs)

| Indicator Type | Value | Context |
| :--- | :--- | :--- |
| **Attacker IPv4** | `10.0.2.4` | Source of Nmap scans, SMB file upload, and reverse shell listener (`TCP/4443`). |
| **Target IPv4** | `10.0.2.15` | Compromised TechNova Systems IIS web server. |
| **C2 Domain (FQDN)** | `cp8nl.hyperhost.ua` | External Command-and-Control domain contacted by AgentTesla. |
| **Malicious Web Shell** | `shell.aspx` | ASPX web shell uploaded via `\\10.0.2.15\Documents`. |
| **Persistence Binary** | `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\updatenow.exe` | UPX-packed AgentTesla executable (`PID 900`) spawned by `w3wp.exe` (`PID 4332`). |
| **SHA-256 Hash** | `C25A6673A24D169DE1BB399D226C12CDC666E0FA534149FC9FA7896EE61D406F` | File hash for `updatenow.exe` (AgentTesla). |
| **Network Signature** | Outbound `TCP/4443` from `w3wp.exe` (`PID 4332`) | Reverse shell callback channel. |
