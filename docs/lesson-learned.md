# Lessons Learned & Hardening Recommendations

## Root Cause Analysis
1. **Exposed SMB Shares on a Public-Facing Server:** Allowing SMB file-sharing access (`TCP/445`) to a web-accessible directory (`Documents`) enabled the attacker to write an executable `.aspx` web shell directly onto the IIS server.
2. **Unrestricted Outbound Egress Traffic:** The web server was permitted to initiate arbitrary outbound connections on non-standard ports (`TCP/4443`) and resolve external domains (`cp8nl.hyperhost.ua`), allowing both the reverse shell and AgentTesla C2 channel to succeed.
3. **Lack of Process Execution Guardrails:** IIS worker process `w3wp.exe` was able to write to and execute binaries inside the system-wide `StartUp` folder without triggering Attack Surface Reduction (ASR) blocks.

---

## Long-Term Architectural Hardening

| Control Area | Actionable Recommendation | Mitigated ATT&CK Technique |
| :--- | :--- | :--- |
| **Network Segmentation & Firewall** | Block inbound SMB (`TCP/139`, `TCP/445`) on all public-facing servers and enforce strict **egress filtering** so web servers can only initiate outbound traffic to approved internal backends and update repositories. | `T1021.002` (SMB Shares) & `T1571` (Non-Standard Port) |
| **IIS & Web Server Hardening** | Separate file storage shares from IIS executable web roots, disable `.aspx` script execution in upload directories, and deploy a Web Application Firewall (WAF) to rate-limit Nmap/NSE scanning probes. | `T1505.003` (Web Shell) |
| **Endpoint Hardening (ASR Rules)** | Enable Microsoft Defender **Attack Surface Reduction (ASR)** rules to *block Office/web server processes (`w3wp.exe`) from creating child processes* and lock down write permissions on `ProgramData\...\StartUp`. | `T1059` (Execution) & `T1547.001` (Startup Folder) |
| **Detection Engineering** | Create high-severity SIEM correlation rules alerting whenever `w3wp.exe` spawns any child process or initiates outbound network connections to external IPs. | `T1505.003` & `T1071.001` |
