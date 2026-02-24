# OpenClaw Agentic Operating System Master Index

## 1. Purpose
This document is the control index for the OpenClaw operational document set. It shows how Discord, LLM governance, and Obsidian memory work together as one system.

### 1.1 Executive Summary (One-Page)
The OpenClaw Agentic Operating System is a three-plane operating model:
- **Control Plane (Discord):** workflow intake, approvals, and operator visibility.
- **Policy Plane (LLM Governance):** model routing, risk gates, and budget controls.
- **Memory Plane (Obsidian):** durable records, audit trails, and recovery evidence.

OpenClaw is the system orchestration runtime: it turns governed intent into executed actions while enforcing policy and preserving an auditable memory trail.

How it works in practice:
1. A request enters through a policy-approved Discord channel.
2. LLM routing selects model class and enforces cost/security constraints.
3. OpenClaw executes tasks and logs structured metadata.
4. Obsidian stores artifacts and references for replay, audit, and recovery.
5. Discord returns status, outputs, and trace links.

#### Example Scenario: Product Campaign Research to Action
1. The operator asks in **#prd-widget-marketing**: "Find 3 competitor campaigns this week and propose one test we can run."
2. OpenClaw classifies the request as product research and applies internet tier controls (R2 by default for product marketing).
3. LLM policy selects model class and token budget, then runs staged retrieval (search -> triage -> synthesis).
4. OpenClaw posts a concise recommendation in Discord with source citations and confidence notes.
5. OpenClaw writes a structured artifact to Obsidian with **request_id**, **project_id**, sources, rationale, and next action.
6. The operator approves the suggested test, and the resulting action is logged in change records for review.

### 1.2 Cost Control Summary
Primary cost controls used by this AOS:
- Tiered model routing (Class A/B/C) by workflow type.
- Budget caps at request, workflow/channel, and project levels.
- Escalation only on quality/complexity triggers.
- Context hygiene (summaries over raw history) to limit token burn.
- Fallback/degraded mode to stop runaway retries and spend.
- Weekly KPI review for cost per successful workflow and escalation rate.

### 1.3 Security Protocol Summary
Primary security protocols used by this AOS:
- Gateway loopback-first exposure model with tunnel/tailnet access.
- Discord DM pairing default with allowlist enforcement.
- Least-privilege bot and role permissions.
- Sandbox policy for shared/non-main execution contexts.
- Trust-boundary split when operators are mutually untrusted.
- Monthly security audit (**openclaw security audit --deep**) and remediation tracking.

### 1.4 Small Business Support Examples
Example ways this AOS supports a small business:
- **Bookkeeping Operations:** ingest receipts, reconcile transactions, and publish month-end summaries into Obsidian with traceable request IDs.
- **Product Marketing/Sales:** run product-scoped campaign channels, generate content drafts, and report KPI deltas with cost-controlled model routing.
- **Research Pipeline:** capture source links, synthesize findings with citations, and evolve ideas through staged gates from hypothesis to execution.

## 2. Core Documents
- [OpenClaw Agentic Operating System Build Procedure](OpenClaw-Agentic-Operating-System-Build-Procedure.md)
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md)
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md)
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md)
- [OpenClaw Skills Specification and Readiness (Customized for the OpenClaw Agentic Operating System)](OpenClaw-Skills-for-Agentic-Operating-System.md)
- [OpenClaw Agentic Operating System Security Validation Runbook](Security-Validation-Runbook.md)

## 2.1 Project Onboarding
- [AOS Project Startup Guide](Project-Startup-Guide.md)

## 3. Interoperability Summary
### 3.1 System Handshake
1. Discord receives a request in the correct channel family.
2. LLM policy routes the request by workflow domain, risk, and budget.
3. OpenClaw executes and logs structured metadata.
4. Obsidian stores durable artifacts and recovery evidence.
5. Discord posts status/result references for operators.

### 3.2 Shared Control Principles
- Least privilege for users and bot permissions.
- Channel isolation by project/workflow to prevent context bleed.
- Model routing and escalation with cost guardrails.
- Markdown-first persistence with metadata for retrieval and recovery.
- Restore drills and audit logs as mandatory operations.

### 3.3 Security Baseline (Cross-Doc)
- Gateway bind remains loopback-only by default.
- Remote access uses SSH tunnel or tailnet controls.
- DM policy defaults to pairing with allowlist approval flow.
- Shared/non-main channel execution uses sandbox profile.
- Trust boundaries are split by gateway/host when operators are mutually untrusted.
- Monthly **openclaw security audit --deep** review is required.
- Internet research access follows tiered controls and context/citation requirements in the LLM policy.

## 4. Shared Data Contract (Cross-Doc)
Use these fields consistently across Discord logs, LLM telemetry, and Obsidian notes:
- **request_id**
- **project_id**
- **workflow_domain**
- **owner** or **requester**
- **status**
- **timestamp_utc**
- **model_class** and **model_name** (when applicable)
- **input_tokens** / **output_tokens** / **estimated_cost** (LLM executions)
- **reference** (link to note/runbook/output)

> **Plain-English Note:** Using the same core fields across systems makes tracing and recovery much faster.

## 5. Naming and Path Standards
### 5.1 Discord Channel Naming
- Product channels: **#prd-<project_id>-marketing|sales|content|analytics**
- Bookkeeping channels: **#bk-<project_id>-inbox|reconcile|reporting**
- Research channels: **#res-<project_id>-inbox|topic-<topic>|syntheses**
- Control channels: **#ctl-openclaw-***

### 5.2 Obsidian Vault Mapping (v1)
- Idea artifacts: **02-Ideas/<project_id>/**
- Research artifacts: **08-Research/<year>/<project_id>/**
- Operational change logs: **07-Operations/Change-Records/<project_id>/**
- Recovery drill evidence: **07-Operations/Recovery-Drills/**

### 5.3 Internet Research Tier Quick Map
| Channel Family | Default Tier | Notes |
|---|---|---|
| **#bk-*** | R1 | Metadata/snippet-level retrieval only |
| **#prd-*-analytics** | R1 | Low-cost, bounded data retrieval |
| **#prd-*-marketing|sales|content** | R2 | Controlled full-page retrieval |
| **#res-*** | R2 | Deep research by default |
| **#res-*** (elevated) | R3 | Requires explicit Owner approval |
| **#ctl-*** | Restricted | Policy-limited retrieval only |

## 6. RACI Snapshot (Who Owns What)
| Area | Owner | Operator | Reviewer |
|---|---|---|---|
| Discord channel policy | Owner | Operator | Reviewer |
| LLM routing and budgets | Owner | Operator | Reviewer |
| Obsidian metadata and retention | Owner | Operator | Reviewer |
| Incident response execution | Owner | Operator | Reviewer |
| Recovery drill governance | Owner | Operator | Reviewer |

## 7. Review Cadence
- Daily: alerts, budget anomalies, failed runs.
- Weekly: KPI and cost review, blocked workflow review.
- Monthly: restore drill and policy adjustments.
- Quarterly: channel cleanup, model portfolio review, disaster drill.

## 8. Change Management Flow
1. Propose change in Discord change-review channel.
2. Assess risk, cost impact, and rollback plan.
3. Update affected docs (Discord/LLM/Obsidian/Procedure).
4. Execute controlled test.
5. Approve and promote to active.
6. Log outcome and references in Obsidian change records.

## 9. Interoperability Health Checklist
- [ ] Channel naming follows the **project_id** pattern.
- [ ] Each active channel family maps to an Obsidian path.
- [ ] Each workflow domain maps to a default LLM class.
- [ ] Escalation and fallback are configured and tested.
- [ ] Required shared metadata fields appear in logs/notes.
- [ ] Monthly restore drill completed with evidence.
- [ ] Gateway loopback and remote-access hardening are validated.
- [ ] DM pairing policy and allowlists are active and tested.
- [ ] Trust-boundary and sandbox policies are validated for shared channels.
- [ ] Monthly security audit findings are logged and remediated.

## 10. Current Status Notes
- Document orchestration links exist across Discord, LLM, and Obsidian guides.
- Discord and Obsidian path placeholders have been aligned to **project_id** and vault v1 structure.
- Remaining work is execution readiness (creating starter MOCs/notes and running first drill).

## 11. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.1.8 | Added AOS Project Startup Guide for standardized new-project onboarding |
| 2026-02-23 | 1.1.7 | Renamed document title to "OpenClaw Agentic Operating System Master Index" for project-name consistency |
| 2026-02-23 | 1.1.6 | Added explicit OpenClaw orchestration-runtime role statement to Executive Summary |
| 2026-02-23 | 1.1.5 | Added concrete "How it works in practice" scenario for product campaign research-to-action flow |
| 2026-02-23 | 1.1.4 | Added executive summary with operating model, cost controls, security protocol overview, and small-business support examples |
| 2026-02-23 | 1.1.3 | Added compact R1/R2/R3 internet research tier mapping table for operator quick reference |
| 2026-02-23 | 1.1.2 | Added Master Index reference to LLM Internet Research Policy controls |
| 2026-02-23 | 1.1.1 | Added Security Validation Runbook to core document set |
| 2026-02-23 | 1.1.0 | Security Hardening v1: added cross-doc security baseline and enforcement checklist items |
| 2026-02-23 | 1.0.0 | Initial master index: interoperability map, shared contract, standards, and review cadence |
