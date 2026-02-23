# Obsidian Roles and Tasks in the OpenClaw Architecture

## 1. Purpose
This document explains what Obsidian does in the OpenClaw system and how to operate it safely and consistently.

> **Plain-English Note:** Obsidian is the long-term memory workspace for OpenClaw. It stores useful outputs so work is searchable and reusable later.

## 2. Obsidian's Role in the System
In this architecture, Obsidian functions as:
- **Knowledge Vault:** Stores research notes, summaries, and reusable references.
- **Workflow Archive:** Keeps records of what OpenClaw did and when.
- **Operational Journal:** Tracks incidents, troubleshooting notes, and decisions.
- **Recovery Support:** Helps reconstruct state after service interruptions.

## 3. What Should Be Written to Obsidian
OpenClaw should write or sync:
- Final research summaries
- Task execution logs (human-readable)
- Runbook updates and decision notes
- Incident reports and postmortems
- Prompt templates and approved system personas

OpenClaw should not write:
- Raw secrets (`.env` values, tokens, private keys)
- Full sensitive logs with credentials
- Unfiltered private personal data

### 3.1 Cross-Document Orchestration
This Obsidian guide is part of a coordinated operating set:
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md) defines channel workflows and command governance.
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md) defines model routing, escalation, and spend control.

Obsidian responsibilities in the unified system:
1. Persist normalized outputs from Discord workflows.
2. Store execution evidence, decision notes, and recovery artifacts.
3. Provide retrieval-ready records for audits, rebuilds, and drill tests.

> **Plain-English Note:** Obsidian is the durable memory layer that keeps decisions and workflow history recoverable.

### 3.2 Workspace Memory Trust Boundary
- Vault memory files are trusted operator state, not a security sandbox.
- If an actor can edit vault memory files, they are inside the trusted boundary.
- Isolation between untrusted operators requires separate OS user/host or separate gateway deployment.

## 4. Recommended Vault Structure
Use a stable folder layout so automation can target known paths.

```text
Vault/
  Operations/
    Daily-Logs/
    Incident-Reports/
    Change-Records/
  Research/
    2026/
    Topics/
  OpenClaw/
    Prompts/
    Workflows/
    Skill-Design/
  Reference/
    Architecture/
    SOPs/
```

## 5. Core Obsidian Task Workflows

### 5.1 Research Capture Workflow
1. User requests research in Discord.
2. OpenClaw gathers and summarizes findings.
3. OpenClaw writes a markdown report into `Research/<year>/`.
4. OpenClaw posts a completion message with file path.

**Output:** A timestamped markdown report with source notes and summary.

### 5.2 Operations Log Workflow
1. OpenClaw completes a major action (deploy, restart, config change).
2. OpenClaw appends a short entry to `Operations/Daily-Logs/`.
3. Entry includes actor, action, result, and follow-up if needed.

**Output:** Auditable daily event trail.

### 5.3 Incident Workflow
1. Failure is detected (bot offline, Ollama unreachable, auth failure).
2. OpenClaw creates incident note in `Operations/Incident-Reports/`.
3. Note captures impact, timeline, root cause, and fix.
4. Post-incident actions are tracked until closed.

**Output:** Repeatable learning and fewer repeated failures.

## 6. Data Quality Rules for Notes
Every automated note should include:
- `Title`
- `Date/Time (UTC)`
- `Trigger/Request`
- `Action Taken`
- `Result`
- `Next Step` (if any)

> **Plain-English Note:** Consistent note format makes searching and troubleshooting much easier.

## 7. Integration Methods
You can use one or both:
- **Obsidian REST API** for direct note writes from OpenClaw.
- **Git-based sync** for version history and rollback.

Recommended pattern:
- Write note -> validate content -> sync/commit -> notify Discord.

## 8. Security and Privacy Guardrails
- Keep API keys outside notes and in `.env` only.
- Redact sensitive values before writing logs.
- Restrict Obsidian API access to trusted network paths (Tailscale/private).
- Use private repositories for vault backups.

## 9. Reliability and Recovery Tasks
- Run scheduled backups (local + offsite).
- Test restore monthly from backup.
- Keep note writes idempotent (safe to retry without duplicates where possible).
- If write fails, queue retry and alert in Discord.

## 10. Operating Checklist
- [ ] Vault folder structure exists and is stable.
- [ ] OpenClaw write targets are configured.
- [ ] Obsidian API access is restricted.
- [ ] Backup schedule is active.
- [ ] Restore test was completed in the last 30 days.
- [ ] Incident template exists and is in use.

## 11. Quick Example Note Template
```markdown
# Operation Log - <YYYY-MM-DD>
- Time (UTC):
- Trigger:
- Action:
- Result:
- Related System: OpenClaw / Ollama / Discord / Obsidian
- Follow-up:
```

## 12. Vault Specification v1 (Local-First Knowledge Graph)

This section defines a machine-ready Obsidian design so the vault can act as an AI disaster-recovery blueprint.

### 12.1 Design Goals
- Fast capture without losing structure
- Reliable retrieval by humans and automation
- Semantic links between ideas, SOPs, prompts, and systems
- Recovery-first documentation for rebuilding workflows after failures

### 12.2 Folder and Tag Hierarchy
Use this structure as the default operating layout.

```text
Vault/
  00-Inbox/
  01-MOCs/
  02-Ideas/
  03-SOPs/
  04-Prompts/
  05-Systems/
  06-Data-Schemas/
  07-Operations/
    Daily-Logs/
    Incident-Reports/
    Change-Records/
    Recovery-Drills/
  08-Research/
    2026/
  09-Archive/
```

Tag model (examples):
- Domain tags: `#domain/openclaw`, `#domain/ops`, `#domain/research`
- Type tags: `#type/idea`, `#type/sop`, `#type/prompt`, `#type/system`, `#type/schema`
- Status tags: `#status/draft`, `#status/active`, `#status/deprecated`
- Criticality tags: `#tier/t1`, `#tier/t2`, `#tier/t3`

> **Plain-English Note:** Tags should be short and predictable so search and Dataview queries stay clean.

### 12.3 YAML Frontmatter Contract (Required Fields)
Use this property schema on all operational notes.

Common required fields:
- `id`
- `type`
- `title`
- `status`
- `owner`
- `created_utc`
- `updated_utc`
- `tags`
- `related_systems`
- `dependencies`
- `inputs`
- `outputs`
- `recovery_tier`

Optional but recommended:
- `runbook_ref`
- `prompt_ref`
- `schema_ref`
- `last_tested_utc`
- `review_cycle_days`

### 12.4 Note-Type Templates (Machine-Ready)

#### Business Idea Note (`02-Ideas/`)
```yaml
---
id: IDEA-0001
type: idea
title: Discord Incident Summarizer
status: draft
owner: you
created_utc: 2026-02-23T00:00:00Z
updated_utc: 2026-02-23T00:00:00Z
tags: ["type/idea", "domain/openclaw", "status/draft"]
related_systems: ["OpenClaw", "Discord", "Obsidian"]
dependencies: ["SOP-1001", "PRM-2001"]
inputs: ["incident-log"]
outputs: ["incident-summary-note"]
recovery_tier: t2
runbook_ref: "[[SOP-1001 Incident Response]]"
prompt_ref: "[[PRM-2001 Incident Summary Prompt]]"
schema_ref: "[[SCH-3001 Incident Note Schema]]"
review_cycle_days: 30
---
```

#### SOP Note (`03-SOPs/`)
```yaml
---
id: SOP-1001
type: sop
title: Incident Response and Recovery
status: active
owner: operations
created_utc: 2026-02-23T00:00:00Z
updated_utc: 2026-02-23T00:00:00Z
tags: ["type/sop", "domain/ops", "status/active", "tier/t1"]
related_systems: ["OpenClaw", "Ollama", "Discord"]
dependencies: ["SYS-4001", "SCH-3001"]
inputs: ["alert", "service-log"]
outputs: ["recovery-action", "postmortem-note"]
recovery_tier: t1
last_tested_utc: 2026-02-20T00:00:00Z
review_cycle_days: 14
---
```

#### Prompt Template Note (`04-Prompts/`)
```yaml
---
id: PRM-2001
type: prompt
title: Incident Summary Prompt
status: active
owner: operations
created_utc: 2026-02-23T00:00:00Z
updated_utc: 2026-02-23T00:00:00Z
tags: ["type/prompt", "domain/openclaw", "status/active"]
related_systems: ["OpenClaw", "Ollama"]
dependencies: ["SCH-3001"]
inputs: ["incident-log", "timeline"]
outputs: ["markdown-summary"]
recovery_tier: t2
schema_ref: "[[SCH-3001 Incident Note Schema]]"
review_cycle_days: 30
---
```

#### System Note (`05-Systems/`)
```yaml
---
id: SYS-4001
type: system
title: OpenClaw Gateway on VPS
status: active
owner: platform
created_utc: 2026-02-23T00:00:00Z
updated_utc: 2026-02-23T00:00:00Z
tags: ["type/system", "domain/openclaw", "status/active", "tier/t1"]
related_systems: ["Discord", "Ollama", "Obsidian"]
dependencies: ["SOP-1001"]
inputs: ["discord-command"]
outputs: ["llm-response", "operation-log"]
recovery_tier: t1
runbook_ref: "[[OpenClaw Agentic Operating System Build Procedure (Discord-Only)]]"
review_cycle_days: 14
---
```

### 12.5 MOC Strategy (Manual Index Notes)
Create a MOC for each major domain:
- `01-MOCs/MOC-OpenClaw-Architecture.md`
- `01-MOCs/MOC-Operations.md`
- `01-MOCs/MOC-Prompts-and-Automation.md`
- `01-MOCs/MOC-Recovery-Readiness.md`

Each MOC should contain:
- Scope statement
- Key links (`[[...]]`) to Ideas, SOPs, Prompts, Systems, Schemas
- "Current Priority" block
- "Gaps / Missing Docs" block

> **Plain-English Note:** MOCs are manual index pages. They help you navigate quickly even if automated queries fail.

### 12.6 Internal Linking Pattern
Use explicit links in note bodies to map logic chains:
- Idea -> SOP: `[[SOP-1001 Incident Response and Recovery]]`
- SOP -> Prompt: `[[PRM-2001 Incident Summary Prompt]]`
- SOP -> Schema: `[[SCH-3001 Incident Note Schema]]`
- SOP -> System: `[[SYS-4001 OpenClaw Gateway on VPS]]`

Relationship sentence pattern:
- `This workflow executes through [[SYS-4001 OpenClaw Gateway on VPS]] using prompt [[PRM-2001 Incident Summary Prompt]] and writes output in schema [[SCH-3001 Incident Note Schema]].`

### 12.7 Dataview Queries (Self-Indexing Views)

#### Query A: Active SOPs by Tier
```dataview
TABLE id, status, recovery_tier, owner, updated_utc
FROM "03-SOPs"
WHERE type = "sop" AND status = "active"
SORT recovery_tier ASC, updated_utc DESC
```

#### Query B: Recovery-Critical Items Missing Recent Test
```dataview
TABLE id, title, last_tested_utc, review_cycle_days
FROM "03-SOPs" OR "05-Systems"
WHERE recovery_tier = "t1" AND (date(today) - date(last_tested_utc)).days > review_cycle_days
SORT updated_utc ASC
```

#### Query C: Dependency Map for a Workflow
```dataview
TABLE id, type, dependencies, related_systems
FROM "02-Ideas" OR "03-SOPs" OR "04-Prompts" OR "05-Systems" OR "06-Data-Schemas"
WHERE contains(dependencies, "SOP-1001") OR id = "SOP-1001"
SORT type ASC
```

#### Query D: Notes Missing Required Metadata
```dataview
TABLE id, file.name, status, owner
FROM "02-Ideas" OR "03-SOPs" OR "04-Prompts" OR "05-Systems"
WHERE !id OR !type OR !status OR !owner OR !recovery_tier
SORT file.name ASC
```

### 12.8 Documentation-First Operating Workflow
1. Capture raw thought in `00-Inbox`.
2. Classify note type (Idea/SOP/Prompt/System/Schema).
3. Add required YAML fields.
4. Add internal links to related artifacts.
5. Move note into its target folder.
6. Validate with Dataview missing-fields query.
7. Update relevant MOC index pages.
8. Mark status (`draft` -> `active`) after review.

### 12.9 Governance Rules
- Naming: `<TYPE>-<ID> <Short Title>` for operational notes.
- Status lifecycle: `draft` -> `active` -> `deprecated` -> `archived`.
- Required review cadence:
  - Tier 1: every 14 days
  - Tier 2: every 30 days
  - Tier 3: every 90 days
- No unreviewed Tier 1 note may stay `draft` more than 7 days.

### 12.10 Recovery Drill Procedure (AI Disaster Recovery Test)
Run monthly and after major architecture changes.

Scenario:
- Assume external AI tool context is wiped.
- Rebuild one target workflow using Obsidian only.

Steps:
1. Select one Tier 1 workflow (example: incident summary pipeline).
2. Use MOC + Dataview to identify required SOP, Prompt, Schema, and System notes.
3. Reconstruct execution order from links and metadata.
4. Run one end-to-end test.
5. Record gaps and corrective actions in `07-Operations/Recovery-Drills/`.

Pass/Fail Criteria:
- **Pass** when all are true:
  - Workflow rebuilt with no external undocumented knowledge
  - Required artifacts found in under 15 minutes
  - End-to-end test succeeds
  - Gaps logged with owners and due dates
- **Fail** if any required artifact is missing, stale, or ambiguous enough to block rebuild.

### 12.11 Minimum Deliverables Checklist
- [ ] Folder hierarchy implemented
- [ ] YAML contract enforced on new operational notes
- [ ] 4 MOCs created and linked
- [ ] Dataview queries added and returning results
- [ ] At least 5 notes created (Idea, SOP, Prompt, System, Schema)
- [ ] First recovery drill completed and logged

### 12.12 Backup and Restore Policy (Integrated Recommendations)
Treat this as a mandatory operational policy for vault durability.

#### 12.12.1 Baseline Principle
- Follow the **3-2-1 Rule**:
  - 3 copies of data
  - 2 different storage types
  - 1 offsite copy

#### 12.12.2 Backup Stack (Trio Model)
Use all three layers together:
1. **Sync Layer:** Obsidian Sync (or equivalent) for multi-device continuity.
2. **Version Layer:** Obsidian Git for historical rollback and audit trail.
3. **Archive Layer:** Local Backup plugin (ZIP snapshots) to secondary storage, with offsite replication.

Optional hardening:
- Monthly offline external-drive snapshot kept disconnected except during backup operations.

> **Plain-English Note:** Sync helps you keep devices in step. Backup helps you recover from bad changes, corruption, or deletion.

#### 12.12.3 Target Objectives
Define service objectives for recovery planning:
- **RPO (Recovery Point Objective):** target max data loss = 30 minutes
- **RTO (Recovery Time Objective):** target max restore time = 60 minutes

If your workflow becomes more critical, reduce both targets.

#### 12.12.4 Cadence and Scheduling
- Git auto-backup: every 15-30 minutes
- Local ZIP backup: daily
- Offsite replication verification: daily
- Restore drill: monthly
- Full "rebuild from vault only" disaster drill: quarterly

#### 12.12.5 Retention Policy
- Keep 30 daily snapshots
- Keep 12 monthly snapshots
- Keep 4 quarterly snapshots

Archive older snapshots to cold storage if needed.

#### 12.12.6 Security Controls for Backups
- Use private repositories for Git backups.
- Encrypt offsite archives.
- Keep backup credentials outside notes and in `.env`/secret manager only.
- Restrict backup write paths to trusted hosts.

#### 12.12.7 Restore Test Procedure
Run this monthly:
1. Select latest backup set (sync + git + ZIP).
2. Restore vault to a clean test location.
3. Verify MOCs, Dataview queries, and linked notes resolve correctly.
4. Rebuild one Tier 1 workflow from vault docs only.
5. Record evidence and duration in `07-Operations/Recovery-Drills/`.

Pass criteria:
- Restore completes within RTO.
- Data recency is within RPO.
- Tier 1 workflow executes successfully.

Fail criteria:
- Missing notes/links/metadata block rebuild.
- Restore exceeds RTO or data loss exceeds RPO.

#### 12.12.8 Operational Checklist (Backup-Specific)
- [ ] Sync enabled and healthy
- [ ] Git auto-backup configured and pushing
- [ ] Daily ZIP backup configured
- [ ] Offsite replication confirmed
- [ ] Monthly restore drill completed
- [ ] Quarterly full disaster drill completed
- [ ] Last drill report logged with owner and remediation actions

#### 12.12.9 Archive Sprawl Control Policy
Use tiered retention and automated pruning to prevent uncontrolled ZIP growth.

Tiered retention targets:
- **Daily:** keep last 30 daily backups
- **Weekly:** keep last 12 weekly backups
- **Monthly:** keep last 12 monthly backups
- **Yearly:** keep last 3 yearly backups

Pruning safeguards:
- Never delete the most recent successful backup.
- Never prune a period unless at least one newer verified restore point exists.
- Perform prune only after backup creation + integrity check completes.
- Log all prune actions in `07-Operations/Change-Records/`.

Storage hygiene rules:
- Exclude temporary/cache/trash folders from archive scope where safe.
- Compress and deduplicate where tooling supports it.
- Move yearly backups to lower-cost cold storage.
- Trigger storage alert when backup volume exceeds 80% of allocated quota.

Operator checklist:
- [ ] Tiered retention policy is configured in backup tooling
- [ ] Auto-prune runs after successful backup verification
- [ ] Last prune report reviewed and approved
- [ ] Restore point chain remains continuous after pruning
- [ ] Cold-storage copy for yearly backup is verified

## 13. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.3.0 | Security Hardening v1: clarified workspace memory trust boundary and operator isolation requirements |
| 2026-02-23 | 1.2.2 | Added cross-document orchestration links and clarified Obsidian's role in the unified Discord-LLM-Obsidian flow |
| 2026-02-23 | 1.2.1 | Added archive sprawl control policy with tiered retention, prune safeguards, and storage hygiene rules |
| 2026-02-23 | 1.2.0 | Integrated backup recommendations: 3-2-1 policy, trio model, RPO/RTO, cadence, retention, encryption, and restore-drill controls |
| 2026-02-23 | 1.1.0 | Appended Vault Specification v1: hierarchy, YAML contract, MOCs, Dataview queries, governance, and recovery drill criteria |
| 2026-02-23 | 1.0.0 | Initial document: Obsidian roles and task workflows for OpenClaw architecture |
