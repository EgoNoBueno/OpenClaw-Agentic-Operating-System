# AOS Startup Procedure (From Full Power-Down)

## Document Control
- **Owner:**
- **Version:** 1.1.0
- **Last Updated:** 2026-02-24
- **Status:** Draft

## 1. Objective
Bring the OpenClaw Agentic Operating System from fully powered down to fully operational, assuming all build and security procedures were already completed.

> **Plain-English Note:** This is the "morning startup checklist" for the whole system.

## 2. Assumptions
- Initial deployment and hardening are complete.
- Discord bot is already provisioned.
- VPS and local machine were intentionally powered down.
- Required secrets already exist in **.env** files.

### 2.1 Operator Access Standard (Recommended)
- Use one standard browser type for all operator web tasks (Discord admin, Tailscale admin, and VPS provider panel).
- Use one primary operator login identity/credential set per environment where policy allows.
- Keep credentials in an approved password manager and use auto-fill to reduce login friction and credential-entry errors.

> **Plain-English Note:** Standardizing browser and login flow reduces failed sign-ins, saves time during startup, and lowers copy/paste password mistakes.

## 3. Startup Success Criteria
- VPS is online and reachable.
- Tailscale path is healthy between VPS and local host.
- Ollama is running locally and reachable from VPS.
- OpenClaw service is running on VPS.
- Discord bot is online and responds to an allowlisted user command.

## 4. Startup Order (Required)
1. Power on local Windows host.
2. Confirm internet access and sign in.
3. Start/connect Tailscale on local host.
4. Start local inference node (Windows Ollama).
5. Power on VPS.
6. Connect to VPS and confirm Tailscale.
7. Start/verify OpenClaw service on VPS.
8. Run end-to-end Discord validation.

> **Plain-English Note:** This order prevents common failures where the VPS starts first but cannot reach local inference.

## 5. Phase A - Local Host Bring-Up

### Actions
1. Power on Windows host and log in.
2. Start Ollama Desktop (or confirm Ollama background service is running).
3. Start Tailscale and confirm connected state.
4. Confirm Ollama is listening on the expected local endpoint.

### Validation Gate A
- Local Tailscale status is connected.
- Ollama is listening on expected endpoint/port.

### Rollback A
- Reboot local host if local network routing is unstable.
- Restart Tailscale client and retry.
- Restart Ollama process/service.

## 6. Phase B - VPS Bring-Up

### Actions
1. Power on VPS in provider panel.
2. SSH into VPS (prefer alias):
   - **ssh myvps**
   - **Command Breakdown:** Open a secure shell to the VPS using your saved SSH alias.
3. Confirm system state:
   - **uptime**
   - **Command Breakdown:** Show how long the server has been running and current load.
4. Confirm Tailscale is connected on VPS.

### Validation Gate B
- SSH login works.
- VPS Tailscale node is online.
- VPS can reach local host over tailnet.

### Rollback B
- If VPS is unreachable, use provider serial console and check network.
- Restart Tailscale on VPS and re-authenticate if needed.

## 7. Phase C - Inference Path Validation

### Actions
1. From VPS, test Ollama reachability:
   - **curl http://<LOCAL-WINDOWS-TAILSCALE-IP>:11434**
   - **Command Breakdown:** Send a test request to confirm VPS-to-Ollama connectivity.
2. Confirm non-error service response.

### Validation Gate C
- Curl to Ollama endpoint succeeds from VPS.
- No public exposure is introduced.

### Rollback C
- Re-check local firewall scope.
- Re-check Ollama bind mode and Tailscale route.

## 8. Phase D - OpenClaw Service Startup

### Actions
1. On VPS, check OpenClaw service state:
   - **systemctl status <openclaw-service-name>**
   - **Command Breakdown:** Check whether the service is running and view recent status details.
2. If not active, start service:
   - **sudo systemctl start <openclaw-service-name>**
   - **Command Breakdown:** Start the service now with admin privileges.
3. Ensure service starts at boot:
   - **sudo systemctl enable <openclaw-service-name>**
   - **Command Breakdown:** Configure the service to start automatically on reboot.
4. Check logs for startup errors:
   - **journalctl -u <openclaw-service-name> -n 100**
   - **Command Breakdown:** Show the latest 100 log lines for this service to troubleshoot errors.

### Validation Gate D
- Service is active (running).
- Logs show successful startup and provider connectivity.

### Rollback D
- Restart service and re-check logs.
- Restore last known-good config if startup fails.

## 9. Phase E - Discord Operational Validation

### Actions
1. Confirm bot shows online in Discord server.
2. Send one low-risk test command from allowlisted user.
3. Confirm response round-trip:
   - Discord request -> OpenClaw -> Ollama -> Discord response
4. Confirm deny behavior with non-allowlisted user (if appropriate for environment).

### Validation Gate E
- Bot online and responsive.
- Authorized request succeeds.
- Unauthorized request is denied/ignored.

### Rollback E
- If bot offline, verify token/config and restart service.
- If auth behavior is wrong, fix allowlist before production use.

## 10. Startup Checklist (Quick Use)
- [ ] Local host powered on and internet connected
- [ ] Local Tailscale connected
- [ ] Ollama running on expected endpoint
- [ ] VPS powered on and reachable by SSH
- [ ] VPS Tailscale connected
- [ ] VPS can reach Ollama endpoint
- [ ] OpenClaw service active
- [ ] Discord bot online
- [ ] End-to-end command test passed

## 11. Common Startup Issues
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| VPS online but no LLM response | Local Ollama not up | Curl Ollama endpoint from VPS | Start/restart Ollama and verify bind route |
| Bot offline in Discord | OpenClaw service stopped | Service status and recent logs | Start service and fix startup errors |
| SSH works but inference fails | Tailscale path issue | Both node statuses and reachability | Restart/reconnect Tailscale on both sides |
| Command rejected unexpectedly | Allowlist mismatch | Auth/ACL config | Correct allowlist and re-test |

## 12. Post-Startup Note
After startup succeeds, continue normal operations under the existing runbooks and governance documents.

> **Plain-English Note:** Startup complete means the system is ready, not that safety checks can be skipped.

## 13. Revision History
| Date | Version | Author | Change |
|---|---|---|---|
| 2026-02-24 | 1.1.0 | | Migrated startup flow from WSL2-based inference steps to native Windows Ollama startup and validation |
