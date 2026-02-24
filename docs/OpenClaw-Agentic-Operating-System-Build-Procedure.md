# OpenClaw Agentic Operating System Build Procedure (Discord-Only)

## Document Control
- **Owner:**
- **Version:** 1.3.1
- **Last Updated:** 2026-02-24
- **Status:** Draft

## 1. Objective
Deploy and operate an OpenClaw Agentic Operating System instance with:
- OpenClaw gateway on a Linux VPS (always-on)
- Ollama inference on local Windows host
- Discord as the only bot interface
- Private connectivity via Tailscale

Primary orchestration objective:
- Ensure Discord control plane, LLM policy plane, and Obsidian memory plane operate as one coordinated system.

> **Plain-English Note:** Think of this as building a team of computers with different jobs. The VPS is the always-awake "front desk," your local machine does the heavy AI thinking, and Discord is the chat app you use to talk to the system.

## 2. Scope
This SOP covers:
- Initial deployment
- Secure configuration
- Validation gates for each phase
- Ongoing operations and recovery
- Cross-document interoperability with Discord, LLM, and Obsidian governance guides

This SOP does not cover:
- Telegram, WhatsApp, or other messengers
- Public exposure of Ollama or Obsidian endpoints

> **Plain-English Note:** "Scope" means what this guide includes and what it does not. Staying in scope prevents confusion and reduces mistakes.

### 2.1 Linked Operating Documents
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md)
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md)
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md)
- [Master Index](Master-Index.md)

## 3. Target Architecture (Discord-Only)
| Layer | Component | Location | Connectivity |
|---|---|---|---|
| Interface | Discord Bot | Discord Cloud | HTTPS to VPS |
| Gateway | OpenClaw | VPS (Ubuntu 24.04 LTS) | Tailscale to local Windows host |
| Inference | Ollama | Local Windows host | Bound to Tailscale/localhost only |
| Knowledge | Obsidian Vault/API (optional) | Local machine | Tailscale-restricted |

> **Plain-English Note:** "Architecture" is just the map of where each part lives and how they talk to each other.

## 4. Prerequisites

### 4.1 Access
- VPS provider account and deployed Ubuntu 24.04 server
- SSH access to VPS
- Windows admin rights on local machine
- Discord Developer Portal access
- Tailscale account

### 4.1.1 Operator Access Standard (Recommended)
- Use one standard browser type for all operator web tasks (Discord admin, Tailscale admin, and VPS provider panel).
- Use one primary operator login identity/credential set per environment where policy allows.
- Keep credentials in an approved password manager and use auto-fill to reduce login friction and credential-entry errors.

> **Plain-English Note:** Standardizing browser and login flow reduces failed sign-ins, saves time during setup/startup, and lowers copy/paste password mistakes.

> **Plain-English Note:** These are the keys you need before starting. If one key is missing, setup usually stops midway.

### 4.2 Version Baseline
| Component | Required Version |
|---|---|
| Ubuntu (VPS) | 24.04 LTS |
| Node.js | 22.12.0+ LTS |
| pnpm | latest stable |
| OpenClaw | 2026.2.23+ (or latest patched stable) |
| Windows (Local Host) | Windows 11 (current stable) |
| Ollama | latest stable |

> **Plain-English Note:** Matching versions matter because many setup errors happen when tools are too old or incompatible.

### 4.3 Required Secrets
- Discord bot token
- Optional Obsidian REST API key
- Any model/provider API keys if used

Store all secrets in **.env** files only. Never commit secrets.

> **Plain-English Note:** A "secret" is like a password for apps. If leaked, someone else could control your bot or spend your API credits.

### 4.4 VPS Specs (Minimum and Recommended)
| Resource | Minimum | Recommended | Why It Matters |
|---|---|---|---|
| vCPU | 2 vCPU | 4 vCPU | Handles bot logic, background jobs, and traffic spikes. |
| RAM | 4 GB | 8 GB | Prevents slowdowns/crashes when multiple tasks run. |
| Disk (SSD) | 20 GB | 40+ GB | Leaves room for logs, updates, and rollback artifacts. |
| Network | Stable IPv4 + baseline bandwidth | IPv4 + higher throughput plan | Keeps Discord connectivity and command response stable. |

> **Plain-English Note:** If you start with the minimum specs, it should work for light usage. For smoother performance and fewer resource issues, use the recommended specs.

### 4.5 Security Baseline Requirements (OpenClaw-Aligned)
- Keep Gateway bind mode loopback-only by default.
- Use remote access via SSH tunnel or Tailscale Serve with strong auth.
- Do not expose Gateway or canvas surfaces directly to public internet.
- Set Discord DM policy to pairing mode by default; require explicit approvals to open DM policy.
- Enforce per-channel/user allowlists for command execution.
- For shared or less-trusted channels, run non-main sessions in sandboxed mode.
- Treat workspace memory and plugin code as trusted-operator boundary (not sandbox).
- Run **openclaw security audit --deep** after initial setup and on a recurring schedule.

## 5. Input Record (Fill Before Work)
| Item | Value |
|---|---|
| Instance Name | |
| Environment | |
| VPS IPv4 | |
| VPS Hostname | |
| VPS Tailscale IP | |
| Local Windows Tailscale IP | |
| Discord Application ID | |
| Discord Bot User ID | |
| Allowed Discord User ID(s) | |
| Change Window | |

> **Plain-English Note:** Fill this table first so you do not lose important values during setup.

## 6. Phase A - VPS Bootstrap

> **Why this phase matters:** This creates a clean, secure starting point on the cloud server.

### Actions
1. SSH to VPS as root and patch system:
	- **apt update && apt upgrade -y**
	- **Command Breakdown:** Refresh package lists, then install updates automatically if the first command succeeds.
2. Create operational user and grant sudo:
	- **adduser openclaw**
	- **Command Breakdown:** Create a new Linux user named **openclaw**.
	- **usermod -aG sudo openclaw**
	- **Command Breakdown:** Add **openclaw** to the **sudo** group so the user can run admin commands.

> **IMPORTANT:** WRITE DOWN THE NEW USERNAME AND PASSWORD IMMEDIATELY AND STORE THEM IN YOUR APPROVED PASSWORD MANAGER.
3. Install baseline packages:
	- **apt install -y git curl ca-certificates**
	- **Command Breakdown:** Install Git, curl, and trusted TLS certificates with automatic yes to prompts.
4. Install Node.js 22.x and pnpm as **openclaw** user.
	- **Action Rationale:** This sets up the JavaScript runtime and package manager OpenClaw needs to install dependencies and build/run correctly.

#### Detailed Node.js 22.x + pnpm Installation (Ubuntu 24.04)
1. Switch to a fresh **openclaw** login shell:
	- **su - openclaw**
	- **Command Breakdown:** Switch to the **openclaw** account and load its full login environment.

> **Plain-English Note:** Keep a space on both sides of the hyphen in **su - openclaw**. Without the space before **openclaw**, the shell may parse it incorrectly and fail.
2. Install NodeSource repository prerequisites:
	- **sudo apt update**
	- **Command Breakdown:** Refresh package metadata with admin privileges.
	- **sudo apt install -y ca-certificates curl gnupg**
	- **Command Breakdown:** Install security and download tools needed to add and trust the Node.js repository.
3. Add Node.js 22.x repository and install Node.js:
	- **curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -**
	- **Command Breakdown:** Download the NodeSource setup script and run it as admin to add the Node.js 22.x repo.
	- **sudo apt install -y nodejs**
	- **Command Breakdown:** Install Node.js (and npm) from the newly added repository.
4. Enable Corepack and activate pnpm:
	- **corepack enable**
	- **Command Breakdown:** Turn on Corepack so Node can manage package managers like pnpm.
	- **corepack prepare pnpm@latest --activate**
	- **Command Breakdown:** Download the latest pnpm release and set it as the active pnpm version.
5. Verify install:
	- **node -v**
	- **Command Breakdown:** Show the installed Node.js version.
	- **pnpm -v**
	- **Command Breakdown:** Show the installed pnpm version.

> **Plain-English Note:** Corepack ships with modern Node.js and manages pnpm versions reliably without a separate global npm install.

> **Plain-English Note:** **ca-certificates** installs trusted root certificates so HTTPS connections (like package downloads and Git operations) can be validated securely.

### Validation Gate A
- **node -v** returns 22.x
- **pnpm -v** returns installed version
- **id openclaw** shows sudo group membership

> **Plain-English Note:** **id openclaw** prints that user’s group memberships; confirm **sudo** appears in the list.

> **Plain-English Note:** A validation gate is a checkpoint. Don’t continue until each checkpoint passes.

### Rollback A
- If bootstrap fails, snapshot/backup server and reprovision clean Ubuntu 24.04.
- Remove partial user/config and repeat from step 1.

> **Plain-English Note:** "Rollback" means safely going back to a known-good state if something breaks.

## 7. Phase B - Private Network Bridge (Tailscale)

> **Why this phase matters:** Your VPS and home PC are in different networks. Tailscale creates a private tunnel so they can talk securely.

### Actions
1. Install Tailscale on VPS and authenticate.
2. Install Tailscale on Windows host.
3. Record both node IPv4 Tailscale addresses.
4. Enable MagicDNS in the Tailscale web Admin Console (**https://login.tailscale.com/admin/dns**) (recommended).

> **Why Windows host is standard:**
> - **Reliability:** The Windows Tailscale app stays running in the background for predictable reachability.
> - **Operational Simplicity:** One primary local network client is easier to monitor and troubleshoot.
> - **Consistency:** Startup and recovery steps are simpler when local inference and local networking are both managed on Windows.

> **Action Rationale (Quick Why):**
> - Step 1 makes the VPS join your private tailnet so it can talk safely to your local machine.
> - Step 2 gives your local host a stable private route to the VPS and back.
> - Step 3 gives you known-good target addresses for testing, config, and troubleshooting.
> - Step 4 lets you use stable device names instead of changing IPs.

### Validation Gate B
- VPS can reach local machine over Tailscale.
- Local machine can reach VPS over Tailscale.
- No public port-forwarding required for Ollama.

> **Plain-English Note:** If this phase fails, later AI calls fail too, even if OpenClaw is installed correctly.

### Rollback B
- If connectivity is unstable, re-authenticate both nodes and re-check ACL policy.
- Temporarily pause deployment until stable point-to-point routing is restored.

## 8. Phase C - Local Inference Node (Windows + Ollama)

> **Why this phase matters:** This is where the AI model actually runs. The VPS sends requests here for answers.

### Actions
1. Install Ollama on Windows host.
2. Pull required model(s) (example: Llama/Mistral family).
3. Configure Ollama binding policy:
	- Preferred: localhost + controlled forwarding
	- Alternative: Tailscale interface only
4. Configure Windows firewall inbound rule for required local ports, scoped to private/Tailscale profile only.

> **Action Rationale (Quick Why):**
> - Step 1 installs the local AI runtime your VPS will call.
> - Step 2 downloads the actual models used to answer requests.
> - Step 3 controls where Ollama listens so trusted systems can reach it without public exposure.
> - Step 4 allows required traffic while keeping access restricted to safer network scopes.

### Validation Gate C
- From VPS, **curl http://<LOCAL-WINDOWS-TAILSCALE-IP>:11434** returns Ollama service response.

> **Plain-English Note:** This curl check confirms network reachability to Ollama from the VPS; if it fails, check Tailscale route, firewall scope, and Ollama bind settings.
- Inference test call returns model output.
- Endpoint is not exposed publicly.

> **Plain-English Note:** You want the AI endpoint reachable by your trusted server, but hidden from the public internet.

### Rollback C
- Revert bind config to localhost-only.
- Disable firewall inbound rules if unexpected exposure is detected.

## 9. Phase D - Discord Bot Provisioning

> **Why this phase matters:** Discord is your command center. If bot permissions or intents are wrong, messages won’t work.

### Actions
1. Create bot application in Discord Developer Portal.
2. Enable required intents (including Message Content Intent where needed).
3. Invite bot to target server with minimal required permissions.
4. Record bot token and channel IDs in secure **.env** on VPS.
5. Configure user-ID allowlist in OpenClaw logic.

> **Action Rationale (Quick Why):**
> - Step 1 creates the Discord identity OpenClaw uses to interact with your server.
> - Step 2 allows the bot to receive the events/messages needed for command handling.
> - Step 3 limits bot privileges to reduce risk if something is misconfigured.
> - Step 4 stores sensitive values outside source files for safer operations.
> - Step 5 restricts command execution to approved users only.

### Validation Gate D
- Bot appears online in target Discord server.
- Command from allowed user receives expected response.
- Command from non-allowed user is ignored/denied.

> **Plain-English Note:** Testing both allowed and blocked users proves your security rules are working.

### Rollback D
- Revoke and rotate bot token immediately on leak suspicion.
- Disable bot permissions until new token/config is active.

## 10. Phase E - OpenClaw Install and Configure (VPS)

> **Why this phase matters:** This installs the app and connects all parts (Discord, OpenClaw, Ollama) into one working system.

### Actions
1. As **openclaw** user, clone repository:
	- **git clone https://github.com/OpenClaw/OpenClaw.git**
	- **Command Breakdown:** Copy the OpenClaw code from GitHub to your VPS.
2. Install/build:
	- **pnpm install**
	- **Command Breakdown:** Install all project dependencies.
	- **pnpm run build**
	- **Command Breakdown:** Build the project into runnable output files.
3. Run wizard/config with these mandatory choices:
	- Messenger: **Discord only**
	- LLM provider: **Ollama**
	- Ollama endpoint: **http://<LOCAL-WINDOWS-TAILSCALE-IP>:11434**
4. Populate **.env** and verify no secrets in tracked files.
5. Apply security configuration baseline:
	- Keep **gateway.bind** set to loopback.
	- Keep Discord DM policy in pairing mode unless explicitly justified.
	- Configure allowlists for trusted users/channels.
	- Run initial **openclaw security audit --deep** and resolve high-risk findings.

> **Action Rationale (Quick Why):**
> - Step 1 gets the application code onto the VPS.
> - Step 2 installs dependencies and builds runnable artifacts.
> - Step 3 ensures runtime choices match this SOP (Discord + Ollama + private endpoint).
> - Step 4 wires secrets/config values needed at startup.
> - Step 5 applies the baseline protections before production use.

### Validation Gate E
- Startup completes without fatal errors.
- Discord message round-trip works (request -> OpenClaw -> Ollama -> Discord response).
- Logs show successful provider connection and no auth failures.

> **Plain-English Note:** A "round-trip" test confirms the full chain works end-to-end, not just one isolated part.

### Rollback E
- Restore previous known-good config and **.env** backup.
- Re-run wizard/config and validate again.

## 11. Phase F - Service Hardening and Operations

> **Why this phase matters:** Setup is only the beginning. Hardening keeps the system stable and safer over time.

### Actions
1. Create systemd service for OpenClaw with auto-restart policy.
2. Enable service at boot.

3. Configure centralized logging (**journalctl** baseline) and retention policy.
4. Define patch cadence:
	- OS updates
	- Node/pnpm runtime updates
	- OpenClaw repository updates
5. Define backup cadence for Obsidian vault and config snapshots.

> **Action Rationale (Quick Why):**
> - Step 1 keeps the app running and helps automatic recovery after crashes.
> - Step 2 restores service automatically after reboot.
> - Step 3 gives you traceable logs for debugging, incidents, and audits.
> - Step 4 keeps software current and reduces security/stability risk over time.
> - Step 5 ensures you can recover important data and config after failures.

### Validation Gate F
- Service survives reboot.
- Service auto-recovers after process crash.
- Logs are queryable and include timestamps/context.

> **Plain-English Note:** If your service cannot recover from reboot/crash, it is not production-ready.

### Rollback F

- Disable new unit/service and revert to prior known-good unit file.
- Restore previous package/runtime versions if update introduces regressions.

## 12. Acceptance Criteria (Go-Live)
- [ ] Discord-only path verified (no Telegram/other messenger dependencies)
- [ ] VPS and local Windows inference host communicate only through trusted network path
- [ ] Ollama endpoint reachable from VPS and not publicly exposed
- [ ] User allowlist enforcement confirmed
- [ ] Secrets stored only in **.env** and excluded from version control
- [ ] Service persistence and reboot recovery verified
- [ ] Runbook tested end-to-end with one real command flow
- [ ] Gateway bind is loopback-only and remote access is tunnel/tailnet-based
- [ ] Discord DM policy and allowlists validated in production config
- [ ] **openclaw security audit --deep** completed with no unresolved critical findings
- [ ] Sandbox policy for non-main/shared sessions is documented and tested

> **Plain-English Note:** Go-live means "safe to use for real work." Every checkbox should be true first.

## 13. Security Controls Checklist
- [ ] Least-privilege Discord bot permissions
- [ ] Token rotation process documented
- [ ] Tailscale ACLs reviewed
- [ ] Firewall scope restricted to private/Tailscale profile
- [ ] No hard-coded credentials in source
- [ ] Audit log review procedure defined

> **Plain-English Note:** Security is not one setting. It is a group of habits you check repeatedly.

## 14. Troubleshooting Quick Map
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Bot offline in Discord | Token/intent/config error | Service logs + Discord portal settings | Correct token/intents and restart service |
| No LLM responses | Ollama unreachable | **curl** to Ollama endpoint from VPS | Fix bind/firewall/Tailscale route |
| Works then stops after reboot | Service not enabled | **systemctl status** and **is-enabled** | Enable unit and validate boot start |
| Unauthorized command execution | Allowlist misconfigured | Auth/ACL logs | Correct allowed user ID list |

> **Plain-English Note:** Use this table like a diagnosis chart: find the symptom first, then follow the check/fix path.

## 15. Change Management Notes
- Record each change with date, owner, and rollback notes.
- For production, execute changes only in approved change windows.

> **Plain-English Note:** Change management creates accountability and helps teams undo bad changes quickly.

## 16. Mini Glossary (Beginner-Friendly)
| Term | Simple Meaning |
|---|---|
| VPS | A cloud computer that stays online all the time. |
| Gateway | The part that receives requests and routes them to the right place. |
| Inference | The AI model generating an answer from your prompt. |
| Ollama | Software that runs AI models on your machine. |
| Endpoint | A network address another app can call (like **http://ip:port**). |
| Port | A numbered network door used by an app (example: **11434**). |
| Tailscale | A private mesh VPN that securely connects devices. |
| MagicDNS | A Tailscale feature that lets you use names instead of IP numbers. |
| ACL | Access Control List; rules for who can connect to what. |
| **.env** file | A file used to store secrets and config values safely. |
| Token | A secret key that proves your app is allowed to use a service. |
| Allowlist | A list of approved users or systems that are allowed access. |
| systemd | Linux service manager used to auto-start and monitor apps. |
| **journalctl** | Command used to view logs from Linux services. |
| Rollback | Returning to a previous working setup after a failure. |
| Validation Gate | A checkpoint to verify a phase worked before moving on. |

### Linux Command-Line Acronyms
| Acronym/Command | Stands For | Simple Meaning |
|---|---|---|
| SSH | Secure Shell | A secure way to log into another computer from your terminal. |
| sudo | superuser do | Run one command with admin-level permission. |
| apt | Advanced Package Tool | Ubuntu/Debian tool for installing and updating software. |
| systemd | system daemon | Linux system/service manager that starts and monitors services. |
| CLI | Command-Line Interface | Text-based way to control a computer with typed commands. |
| API | Application Programming Interface | A defined way for one program to talk to another. |
| ID | Identifier | A unique value used to recognize a user, bot, or resource. |
| URL | Uniform Resource Locator | A web/network address like **http://host:port**. |

### Command Examples from This SOP
| Command | What It Does | Why You Use It Here |
|---|---|---|
| **apt update && apt upgrade -y** | Refreshes package lists and installs updates. | Makes sure the VPS starts from a patched, safer baseline. |
| **adduser openclaw** | Creates a new Linux user account. | Avoids running the whole bot stack as root. |
| **usermod -aG sudo openclaw** | Adds the **openclaw** user to the sudo group. | Lets that user run admin tasks when needed. |
| **git clone https://github.com/OpenClaw/OpenClaw.git** | Downloads the OpenClaw code repository. | Gets the application onto the VPS for installation. |
| **pnpm install** | Installs project dependencies. | Prepares OpenClaw to build and run. |
| **pnpm run build** | Compiles/builds the project. | Produces the runnable app artifacts. |
| **curl http://<LOCAL-WINDOWS-TAILSCALE-IP>:11434** | Sends a test request to Ollama. | Confirms VPS can reach your local inference service. |
| **systemctl status <service>** | Shows service health and state. | Verifies OpenClaw service is running correctly. |
| **journalctl -u <service>** | Shows logs for one Linux service. | Helps troubleshoot startup/auth/connectivity issues. |
| **ssh <user>@<server-ip>** | Opens a secure remote shell session. | Connects you to the VPS to perform setup and operations. |

## 17. Informational Notes (Optional but Useful)

### 17.1 Backup Safety Rule (3-2-1)
- Keep **3 copies** of important data.
- Use at least **2 different storage types** (for example, local disk and cloud storage).
- Keep at least **1 offsite copy** in case your main machine is unavailable.

> **Plain-English Note:** This reduces the chance of permanent data loss from mistakes, corruption, or hardware failure.

### 17.2 Sync Is Not the Same as Backup
- **Sync** keeps files the same across locations.
- **Backup** keeps older recoverable versions.
- If a bad change is synced everywhere, backup is what lets you restore a good version.

### 17.3 Local Host Availability Reminder
- If your local PC sleeps or shuts down, the VPS cannot reach your local Ollama service.
- For dependable operation windows, keep the local Windows inference host awake and connected.

### 17.4 MagicDNS Practical Note
- Enabling MagicDNS can reduce breakage from changing local network details.
- Name-based access can be easier to maintain than hard-coded IP references.

### 17.5 Backup Method Snapshot
| Method | Main Benefit | Main Limitation |
|---|---|---|
| Obsidian Sync | Simple device-to-device syncing | Not a full versioned backup strategy by itself |
| Obsidian Git | Version history and rollback points | Requires basic Git workflow understanding |
| Local Backup (ZIP/snapshots) | Fast restore from local copies | Local-only copies are not offsite unless replicated |

## 18. Interoperability Requirements (AOS Build)
These controls are required so the full system works smoothly across components.

### 18.1 Shared Metadata Contract
Each executed workflow should carry:
- **request_id**
- **project_id**
- **workflow_domain**
- **owner** or **requester**
- **status**
- **timestamp_utc**
- **reference** (Obsidian artifact or runbook link)

For LLM-executed tasks also include:
- **model_class**
- **model_name**
- **input_tokens**
- **output_tokens**
- **estimated_cost**

### 18.2 Channel and Path Standards
- Discord channel names must use project-scoped patterns from the Discord roles guide.
- Obsidian artifacts must be written to project-scoped paths aligned to the vault specification.
- All high-risk workflows must map to explicit approval paths.

### 18.3 Cross-Plane Execution Handshake
1. Discord intake validates channel and role policy.
2. LLM routing policy selects model class and budget path.
3. OpenClaw executes and records telemetry.
4. Obsidian stores durable artifact with reference ID.
5. Discord posts completion state and reference.

### 18.4 First-Run Integration Test
Before production go-live, run one workflow from each domain:
- Bookkeeping domain test
- Product campaign domain test
- Research domain test

Each test must complete the full handshake and produce valid artifact references.

## 19. AOS Build Readiness Checklist
- [ ] Discord channel families are created with project-scoped naming.
- [ ] LLM routing policy is mapped to each channel family.
- [ ] Obsidian path mapping is configured for each project domain.
- [ ] Shared metadata contract fields appear in logs and notes.
- [ ] Cost budget caps are active (request/workflow/project).
- [ ] Model escalation and degraded mode paths are tested.
- [ ] One end-to-end test passed for bookkeeping, product, and research workflows.
- [ ] Monthly restore drill and quarterly disaster drill schedules are set.

## 20. Revision History
| Date | Version | Author | Change |
|---|---|---|---|
| 2026-02-24 | 1.3.1 | | Set explicit minimum OpenClaw version baseline to 2026.2.23+ for current security advisory coverage |
| 2026-02-24 | 1.3.0 | | Migrated architecture references from WSL2 inference host to native Windows Ollama host across objective, topology, phases, glossary, and validation examples |
| 2026-02-24 | 1.2.7 | | Added explicit rationale for using Windows-host Tailscale over WSL-only in Phase B |
| 2026-02-24 | 1.2.6 | | Added missing rationale for Phase A Step 4 to complete action-level clarity coverage |
| 2026-02-24 | 1.2.5 | | Added clear action-rationale notes for non-obvious steps across Phases B-F |
| 2026-02-24 | 1.2.4 | | Simplified inline command breakdown notes into shorter one-sentence explanations |
| 2026-02-24 | 1.2.3 | | Added training-focused command breakdown notes across command-driven build steps |
| 2026-02-24 | 1.2.2 | | Added detailed step-by-step Node.js 22.x and pnpm installation instructions for the openclaw user |
| 2026-02-24 | 1.2.1 | | Added recommended operator browser/login standard for easier and safer access workflows |
| 2026-02-23 | 1.2.0 | | Security Hardening v1: added trust-boundary controls, loopback/DM baseline, Node 22.12+ floor, audit, and sandbox go-live requirements |
| 2026-02-23 | 1.1.1 | | Renamed procedure to OpenClaw Agentic Operating System Build Procedure and aligned cross-document references |
| 2026-02-23 | 1.1.0 | | Added Agentic Operating System framing, linked operating docs, interoperability requirements, and build readiness checklist |
| 2026-02-22 | 1.0.8 | | Added optional informational notes appendix (backup rule, sync vs backup, WSL2 availability, MagicDNS, backup-method snapshot) |
| 2026-02-22 | 1.0.7 | | Added VPS sizing guidance with minimum and recommended CPU, RAM, and disk specs |
| 2026-02-22 | 1.0.6 | | Applied final style-consistency pass (wording and placeholder formatting) |
| 2026-02-22 | 1.0.5 | | Proofread and corrected minor wording for clarity |
| 2026-02-22 | 1.0.4 | | Added beginner command-examples table for key Linux/OpenClaw commands |
| 2026-02-22 | 1.0.3 | | Added Linux command-line acronym explanations to the mini glossary |
| 2026-02-22 | 1.0.2 | | Added mini glossary for beginners while preserving technical terms |
| 2026-02-22 | 1.0.1 | | Added plain-English explanatory notes for student-friendly readability |
| 2026-02-22 | 1.0.0 | | Rewrote as Discord-only SOP with validation, security, and rollback gates |
