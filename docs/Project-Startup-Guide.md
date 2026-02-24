# AOS Project Startup Guide

## 1. Purpose
Use this guide to start a new project in the OpenClaw Agentic Operating System (AOS) while keeping all core documentation connected and auditable.

## 2. Quick Start (15-Minute Setup)
1. Pick a **project_id** using your naming standard.
2. Open the [Master Index](Master-Index.md) and confirm governance, security, and data-contract requirements.
3. Create one project charter note from [Procedure Template](../templates/Procedure-Template.md).
4. Create Discord channels and Obsidian paths that match **project_id** conventions.
5. Set default LLM routing tier and budget guardrails for the project.
6. Run one small test workflow and log evidence.

## 3. Required Reference Set
Use all documents below as your baseline pack for every new project:
- [OpenClaw Agentic Operating System Build Procedure](OpenClaw-Agentic-Operating-System-Build-Procedure.md)
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md)
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md)
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md)
- [OpenClaw Skills Specification and Readiness (Customized for the OpenClaw Agentic Operating System)](OpenClaw-Skills-for-Agentic-Operating-System.md)
- [OpenClaw Agentic Operating System Security Validation Runbook](Security-Validation-Runbook.md)

## 4. New Project Checklist
### 4.1 Project Definition
- [ ] **project_id** is defined.
- [ ] Owner, Operator, and Reviewer are assigned.
- [ ] Workflow domain is selected (bookkeeping, product, research, etc.).

### 4.2 Control Plane (Discord)
- [ ] Channels follow the **project_id** naming pattern from the Master Index.
- [ ] Bot permissions and role scopes are least-privilege.
- [ ] Escalation and approval channels are ready.

### 4.3 Policy Plane (LLM Governance)
- [ ] Default model class is set by workflow type.
- [ ] Request and project budget caps are set.
- [ ] Escalation and fallback behavior are documented.

### 4.4 Memory Plane (Obsidian)
- [ ] Project folders exist in the correct vault paths.
- [ ] Notes include shared metadata fields (**request_id**, **project_id**, **workflow_domain**, **status**, **timestamp_utc**).
- [ ] Change records and run evidence paths are defined.

### 4.5 Security and Validation
- [ ] Remote access method is hardened (tunnel/tailnet) and gateway remains loopback-first.
- [ ] DM pairing and allowlist policy are configured.
- [ ] A validation run is executed using the Security Validation Runbook.

## 5. Project Documentation Map (Copy/Paste)
Use this section in each project charter to keep references consistent:

| Area | Primary Document | Why It Matters |
|---|---|---|
| Build baseline | [Build Procedure](OpenClaw-Agentic-Operating-System-Build-Procedure.md) | Canonical setup and verification flow |
| Orchestration map | [Master Index](Master-Index.md) | Cross-system contract and standards |
| Team operating model | [Discord Roles and Tasks](Discord-Roles-and-Tasks.md) | Intake, approvals, and operator workflow |
| Model governance | [LLM Roles and Tasks](LLM-Roles-and-Tasks.md) | Routing, budgets, risk, and quality controls |
| Durable records | [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md) | Persistence, traceability, and recovery support |
| Capability readiness | [OpenClaw Skills for Agentic Operating System](OpenClaw-Skills-for-Agentic-Operating-System.md) | Skill inventory and readiness checks |
| Security assurance | [Security Validation Runbook](Security-Validation-Runbook.md) | Hardening, audit cadence, and test evidence |

## 6. Minimum Exit Criteria (Project Ready)
A project is ready for active execution only when:
- All checklist sections in this document are complete.
- A successful test workflow has traceable evidence in Obsidian.
- A Reviewer signs off on governance, budget, and security controls.

## 7. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.0.0 | Initial AOS project startup guide with full cross-doc reference set |
