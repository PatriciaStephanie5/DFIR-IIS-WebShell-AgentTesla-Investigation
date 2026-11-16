# Containment & Response Runbook

## Phase 1: Immediate Containment
1. **Network Quarantine:** Isolate `10.0.2.15` at the virtual switch / firewall layer to terminate active sessions with `10.0.2.4` (`TCP/4443`) and block outbound C2 beacons to `cp8nl.hyperhost.ua`.
2. **Perimeter Blocking:** Add `10.0.2.4` and `cp8nl.hyperhost.ua` to perimeter firewall, DNS sinkhole, and EDR blocklists across all TechNova Systems network segments.
3. **Process Termination:** Terminate malicious child process `updatenow.exe` (`PID 900`) and recycle/stop the compromised IIS application pool (`w3wp.exe`, `PID 4332`).

## Phase 2: Eradication & Credential Rotation
1. **Artifact Removal:** Remove `shell.aspx` from the IIS web root / `Documents` share and delete `updatenow.exe` from `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\`.
2. **Credential Reset (Critical for AgentTesla):** Because AgentTesla targets stored credentials and application config files, immediately rotate:
   * Local administrator and IIS application pool service account passwords on `10.0.2.15`.
   * Database connection strings, API keys, and SMTP/FTP credentials stored in `web.config` or user profiles on the host.
3. **Clean Re-imaging:** Rebuild `10.0.2.15` from a verified clean baseline image before restoring public web traffic.

## Phase 3: Enterprise Threat Hunting
* Query DNS logs and proxy telemetry across all internal hosts for any historical queries to `cp8nl.hyperhost.ua` or outbound connections on `TCP/4443`.
* Scan all Windows endpoints for the SHA-256 hash `C25A6673A24D169DE1BB399D226C12CDC666E0FA534149FC9FA7896EE61D406F` or unauthorized binaries in `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\`.
