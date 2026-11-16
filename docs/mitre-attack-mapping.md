# MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique / Sub-Technique Name | Forensic Evidence |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | `T1595.002` | **Active Scanning: Vulnerability Scanning** | Attacker (`10.0.2.4`) executed Nmap Scripting Engine (`NSE`) HTTP enumeration probes against `10.0.2.15`. |
| **Discovery** | `T1135` | **Network Share Discovery** | SMB2 Tree Connect requests (`smb2.cmd == 3`) enumerated `\\10.0.2.15\IPC$` and `\\10.0.2.15\Documents`. |
| **Lateral Movement** | `T1021.002` | **Remote Services: SMB/Windows Admin Shares** | Attacker wrote `shell.aspx` to the exposed SMB share (`smb2.cmd == 5`). |
| **Persistence** | `T1505.003` | **Server Software Component: Web Shell** | Uploaded malicious ASPX script (`shell.aspx`) into an IIS-accessible directory for remote command execution. |
| **Execution** | `T1059` | **Command and Scripting Interpreter** | IIS worker process `w3wp.exe` (`PID 4332`) executed reverse shell commands and launched child processes. |
| **Defense Evasion** | `T1027.002` | **Obfuscated Files or Information: Software Packing** | Secondary payload `updatenow.exe` was packed using **UPX** to evade static antivirus signatures. |
| **Persistence** | `T1547.001` | **Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder** | Placed `updatenow.exe` in `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\updatenow.exe` for system-wide persistence. |
| **Command and Control** | `T1571` | **Non-Standard Port** | Established outbound reverse shell from `10.0.2.15` to `10.0.2.4` over **`TCP/4443`**. |
| **Command and Control** | `T1071.001` | **Application Layer Protocol: Web Protocols** | AgentTesla payload configured to beacon to external C2 domain **`cp8nl.hyperhost.ua`**. |
| **Credential Access / Collection** | `T1555` / `T1056` | **Credentials from Password Stores / Input Capture** | Final payload classified as **AgentTesla**, a commodity infostealer targeting browser credentials, mail clients, and keystrokes. |
