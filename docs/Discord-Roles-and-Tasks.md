# Discord Roles and Tasks in the OpenClaw Architecture

## 1. Purpose
This document defines how Discord is used in the OpenClaw system, who is allowed to do what, and how tasks should flow safely.

> **Plain-English Note:** Discord is the control room for OpenClaw. People send commands there, and OpenClaw replies with results and status updates.

## 2. Discord's Role in the System
In this architecture, Discord provides:
- **User Interface:** Primary chat-based command input and output.
- **Notification Hub:** Alerts for workflow completion, failures, and system health.
- **Operational Console:** Controlled channel for running approved automation tasks.
- **Audit Context:** Message history for incident review and timeline reconstruction.

## 3. Scope
This document covers:
- Role model for users and bot permissions
- Channel responsibilities and task boundaries
- Command and response workflow
- Security, reliability, and incident procedures

This document does not cover:
- Telegram/WhatsApp bot operations
- Public command interfaces outside approved Discord servers

### 3.1 Cross-Document Orchestration
This Discord guide is part of a coordinated operating set:
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md) defines persistent memory, metadata, and recovery records.
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md) defines model routing, escalation, and cost controls.

End-to-end flow:
1. Discord receives and classifies the request.
2. LLM routing policy selects the model class and budget path.
3. Output and execution evidence are written to Obsidian.
4. Discord posts status/results with reference IDs.

> **Plain-English Note:** Discord is the control surface, LLM policy is the decision engine, and Obsidian is the long-term memory.

## 4. Discord Role Model (Human Roles)

### 4.1 Owner
- Final authority for bot behavior and production changes.
- Approves high-risk automations and permission changes.
- Owns emergency stop/escalation decisions.

### 4.2 Operator
- Executes approved operational commands.
- Monitors alerts and acknowledges incidents.
- Performs checklist-driven recovery actions.

### 4.3 Reviewer
- Reviews outputs for correctness and safety.
- Validates SOP and prompt updates before activation.
- Flags quality issues and drift.

### 4.4 Observer (Read-Only)
- Views status channels and reports.
- Does not execute operational commands.

> **Plain-English Note:** Use least privilege. Most users should not be able to run sensitive commands.

## 5. Bot Permission Model (Discord Bot)
Grant only what is needed for OpenClaw workflows.

Minimum required bot capabilities:
- Read messages in approved channels
- Send messages and replies
- Read message history
- Add reactions (optional for workflow state)

Avoid unless explicitly required:
- Administrator permission
- Broad server management permissions
- Unrestricted channel access

## 6. Portfolio-Grade Channel Architecture (Multi-Project)
For multiple concurrent projects, use channel families so each business area has isolated context.

### 6.1 Global Control Channels
- **#ctl-openclaw-commands** - global command intake
- **#ctl-openclaw-results** - global results summary
- **#ctl-openclaw-alerts** - system alerts only
- **#ctl-openclaw-ops-log** - operational audit log
- **#ctl-openclaw-change-review** - workflow/prompt change reviews

### 6.2 Bookkeeping Channel Family
- **#bk-inbox** - receipts, expenses, and transaction intake
- **#bk-reconcile** - reconciliation tasks and variance checks
- **#bk-reporting** - month-end and KPI finance outputs

### 6.3 Product Campaign Channel Family (Per Product)
Create one set per product to avoid context pollution:
- **#prd-<project_id>-marketing**
- **#prd-<project_id>-sales**
- **#prd-<project_id>-content**
- **#prd-<project_id>-analytics**

Example:
- **#prd-widget-marketing**
- **#prd-widget-sales**
- **#prd-widget-content**
- **#prd-widget-analytics**

### 6.4 Research Channel Family
- **#res-inbox-links** - raw links and source drops
- **#res-topic-<topic>** - deep research by topic
- **#res-syntheses** - final summaries and decisions

### 6.5 Channel Rules
- Commands are accepted only in approved command channels.
- Each channel maps to one workflow type and one output schema.
- High-risk commands require explicit confirmation workflow.
- Alert channels must remain low-noise and actionable.
- Cross-project requests must declare target project explicitly.
- Channels with mutually untrusted operators must use separate gateway trust boundaries.

> **Plain-English Note:** One channel family per project keeps the AI from mixing unrelated tasks.

### 6.6 Model Routing by Channel Class
- Bookkeeping/reconciliation: fast, low-cost model by default.
- Product analytics extraction: fast, low-cost model.
- Research synthesis and strategy: high-capability model.
- Fallback policy: primary model timeout -> secondary model -> degraded response.

### 6.7 Internet Research Tier Mapping (R1/R2/R3)
Use these tiers from the LLM Internet Research Policy to control web access by channel family.

Default mapping:
- **#bk-*** channels -> **R1** (metadata/snippet-level only)
- **#prd-*-analytics** -> **R1**
- **#prd-*-marketing|sales|content** -> **R2** (controlled full-page retrieval)
- **#res-inbox-links**, **#res-topic-***, **#res-syntheses** -> **R2** by default, **R3** only with explicit Owner approval
- **#ctl-*** channels -> no broad research crawling; use policy-limited retrieval only

Operator rules:
- Any R3 request requires explicit approval record in **#ctl-openclaw-change-review**.
- If tier is unspecified in request context, default to the lower tier.
- All research outputs must include source references and confidence notes.

> **Plain-English Note:** R1 is safer and cheaper, R2 is deeper research, and R3 is tightly restricted.

## 7. Core Discord Task Workflows

### 7.1 Standard Command Workflow
1. Authorized user submits command in **#ctl-openclaw-commands**.
2. Bot validates user role and command policy.
3. Bot executes allowed workflow.
4. Bot posts result to **#ctl-openclaw-results** and log summary to **#ctl-openclaw-ops-log**.

### 7.2 High-Risk Action Workflow
1. Operator issues high-risk command.
2. Bot requests confirmation token or secondary approval.
3. Bot logs approver and execution context.
4. Bot executes action and posts completion status.

### 7.3 Incident Workflow
1. Bot detects failure (provider offline, auth error, timeout spike).
2. Bot posts structured alert in **#ctl-openclaw-alerts**.
3. Operator acknowledges and starts runbook steps.
4. Resolution summary is posted in **#ctl-openclaw-ops-log**.

### 7.4 Bookkeeping Workflow
1. User submits bookkeeping task in **#bk-inbox** or **#bk-reconcile**.
2. Bot validates channel policy and requester role.
3. Bot processes task and posts status in the same channel.
4. Bot writes structured finance note to Obsidian and posts reference link.

### 7.5 Product Campaign Workflow
1. User posts campaign task in the product-specific channel family.
2. Bot executes only within that product context.
3. Bot returns result and KPI summary in the same product channel.
4. Bot writes campaign artifact to product-scoped Obsidian path.

### 7.6 Research Workflow
1. User drops sources in **#res-inbox-links** or topic channel.
2. Bot summarizes, classifies, and synthesizes findings.
3. Bot posts digest in **#res-syntheses**.
4. Bot writes markdown-first research output to Obsidian.

## 8. Access and Identity Controls
- Enforce Discord user-ID allowlist for command execution.
- Restrict command execution to approved roles.
- Ignore commands from unapproved users/channels.
- Rotate bot token immediately if compromise is suspected.
- Store token in **.env** only (never in notes or code).

### 8.1 DM Security Policy (Required)
- Default DM policy must be pairing mode.
- Unknown senders should receive pairing flow and no command execution.
- Open DM policy must require explicit Owner approval and change log entry.
- Approved senders must be in allowlist and reviewable via periodic audit.

### 8.2 Trust Boundary Rule
- A single gateway is treated as one trusted operator boundary.
- Do not mix mutually untrusted operator groups on one gateway.
- If trust boundaries differ, split by OS user/host or separate gateway instance.

## 9. Message and Logging Standards
Each operational bot message should include:
- **Request ID**
- **Requester**
- **Project ID** or **Workflow Domain**
- **Command**
- **Status** (started/succeeded/failed)
- **Timestamp (UTC)**
- **Reference** (related note or runbook link)

> **Plain-English Note:** Standard message format makes troubleshooting much faster.

## 10. Reliability and Safety Controls
- Implement command timeout and retry policy.
- Use degraded mode messaging when LLM backend is unavailable.
- Add rate limits for expensive or long-running commands.
- Require explicit confirmation for destructive actions.
- Keep a circuit-breaker command to pause automation safely.
- For shared/non-main channels, enforce sandboxed execution profile.

## 11. Governance and Change Control
- Proposed prompt/workflow changes are reviewed in **#ctl-openclaw-change-review**.
- Production changes require approval from Owner or delegated Reviewer.
- Track change date, approver, rollback plan, and expected impact.
- Re-test critical commands after each major change.

Additional portfolio governance:
- Each channel family must have an owner.
- Each product channel family must define allowed command types.
- New product onboarding requires channel creation + Obsidian path mapping + policy test.
- Quarterly review removes stale channels and archives inactive project lanes.

## 12. Operational Checklist
- [ ] Discord bot permissions are least-privilege.
- [ ] User-ID allowlist is configured and tested.
- [ ] Command channels are restricted correctly.
- [ ] Alert channel is monitored by an Operator.
- [ ] High-risk command confirmation path is enabled.
- [ ] Incident workflow has been tested in the last 30 days.
- [ ] Bookkeeping, product, and research channel families are mapped and owned.
- [ ] Product-specific channels are linked to product-specific Obsidian paths.
- [ ] Model routing policy is defined per channel family.
- [ ] DM policy is pairing by default and documented.
- [ ] Trust boundary review confirms no mutually untrusted users share a gateway.
- [ ] Sandbox profile for shared/non-main channels is enabled and tested.

## 13. Project Onboarding Workflow and Tasks
Use this workflow every time a new project, product line, or campaign portfolio is added.

### 13.1 Required Intake Fields
- **project_id** (short unique key)
- **project_name**
- **owner**
- **business_domain** (bookkeeping, product, research)
- **risk_tier** (t1/t2/t3)
- **allowed_commands**
- **required_outputs**
- **obsidian_path**
- **default_model_class** (fast or high-capability)

### 13.2 Onboarding Workflow
1. Create project record in **#ctl-openclaw-change-review** with all intake fields.
2. Approve channel family plan and role access.
3. Create Discord channels using naming standard.
4. Map each channel to allowed commands and output schema.
5. Create corresponding Obsidian paths and note templates.
6. Configure model routing policy for each channel.
7. Execute policy test commands and confirm logs + Obsidian writes.
8. Mark project as active in **#ctl-openclaw-ops-log**.

### 13.3 Channel Naming Standard
- Product channels: **#prd-<project_id>-marketing|sales|content|analytics**
- Bookkeeping channels: **#bk-<project_id>-inbox|reconcile|reporting**
- Research channels: **#res-<project_id>-inbox|topic-<topic>|syntheses**

### 13.4 Onboarding Task Checklist
- [ ] Project owner assigned
- [ ] Channel set created and access tested
- [ ] Allowed commands list applied
- [ ] Obsidian path and templates created
- [ ] Model routing assigned by channel
- [ ] High-risk command confirmation validated
- [ ] First end-to-end command passed

## 14. Idea Research and Evolution Workflow
This workflow turns raw ideas into executable, testable project actions.

### 14.1 Lifecycle Stages
1. **Capture** - collect raw ideas and links.
2. **Clarify** - rewrite idea into a clear hypothesis.
3. **Validate** - test feasibility, risks, and dependencies.
4. **Plan** - define tasks, KPI targets, and execution path.
5. **Execute** - run campaign/research actions.
6. **Review** - evaluate outcomes against KPIs.
7. **Evolve** - improve, pivot, archive, or scale.

### 14.2 Stage-to-Channel Mapping
- Capture: **#res-inbox-links** or **#prd-<project_id>-content**
- Clarify/Validate: **#res-topic-<topic>**
- Plan: **#prd-<project_id>-marketing** or **#prd-<project_id>-sales**
- Execute: relevant product/bookkeeping operational channel
- Review/Evolve: **#res-syntheses** + **#ctl-openclaw-results**

### 14.3 Task Outputs Required at Each Stage
- Capture output: idea note with source links and tags.
- Clarify output: one-sentence hypothesis + assumptions.
- Validate output: risk list, dependencies, go/no-go decision.
- Plan output: task breakdown, owner, due date, KPI definitions.
- Execute output: action log + generated artifacts.
- Review output: KPI delta, lessons learned, next-step decision.
- Evolve output: updated playbook or archived rationale.

### 14.4 Decision Gates
- **Gate A (Promote to Plan):** hypothesis is specific and measurable.
- **Gate B (Promote to Execute):** dependencies ready and risk acceptable.
- **Gate C (Promote to Scale):** KPI target met in at least one test cycle.
- **Gate D (Archive):** repeated underperformance or strategic deprioritization.

### 14.5 Obsidian Integration for Idea Evolution
For every stage, write a markdown artifact to project-scoped vault path:
- **08-Research/<year>/<project_id>/**
- **02-Ideas/<project_id>/**
- **07-Operations/Change-Records/<project_id>/**

Each artifact should include:
- **project_id**
- **idea_id**
- **stage**
- **owner**
- **kpi_target**
- **result_summary**
- **next_stage**

> **Plain-English Note:** This gives you an auditable trail from first idea to real-world result.

### 14.6 Weekly Portfolio Review Tasks
- Review all active ideas by stage and owner.
- Identify blocked items and unresolved dependencies.
- Re-route model class if quality/speed is off-target.
- Retire stale ideas with archive notes.
- Promote top performers into repeatable SOP candidates.

## 15. Quick Incident Message Template
```markdown
[OpenClaw Alert]
- Severity:
- Request ID:
- Detected At (UTC):
- Component: Discord / OpenClaw / Ollama / Obsidian
- Symptom:
- Immediate Action:
- Next Update ETA:
```

## 16. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.3.1 | Added R1/R2/R3 internet research tier mapping by channel family and operator approval rules |
| 2026-02-23 | 1.3.0 | Security Hardening v1: added DM pairing policy, trust-boundary rule, and sandbox controls for shared channels |
| 2026-02-23 | 1.2.2 | Standardized **project_id** channel placeholders and aligned Obsidian integration paths with vault specification v1 |
| 2026-02-23 | 1.2.1 | Added cross-document orchestration links and end-to-end operating flow with Obsidian and LLM governance docs |
| 2026-02-23 | 1.2.0 | Added project onboarding workflow/template and idea research/evolution lifecycle with task gates |
| 2026-02-23 | 1.1.0 | Added portfolio-grade multi-project channel architecture, channel-family workflows, and model routing guidance |
| 2026-02-23 | 1.0.0 | Initial Discord roles and tasks document for OpenClaw architecture |
