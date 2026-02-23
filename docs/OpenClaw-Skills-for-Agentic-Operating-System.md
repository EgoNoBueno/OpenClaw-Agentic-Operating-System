# OpenClaw Skills Specification and Readiness (Customized for the OpenClaw Agentic Operating System)

## 1. Purpose
This document defines how to design, validate, and operate OpenClaw skills so they align with the OpenClaw Agentic Operating System (AOS).

> **Plain-English Note:** A skill is one reusable job OpenClaw can do reliably. This guide makes sure skills are safe, testable, and cost-aware in your AOS.

## 2. Scope
This document covers:
- Skill contract template
- AOS-specific policy requirements
- Skill readiness checklist
- Rollout and rollback process
- Telemetry and quality controls

This document does not cover:
- Low-level OpenClaw source code internals
- Vendor-specific API billing setup

## 3. AOS Customization Principles
Every skill must align with all three AOS planes:
- **Discord Control Plane:** where and by whom the skill can be invoked.
- **LLM Policy Plane:** what model class, budget, and escalation rules apply.
- **Obsidian Memory Plane:** what artifacts and evidence are written for recovery.

Hard requirements:
- One skill = one primary business function.
- Structured inputs and outputs only.
- Idempotent execution using `request_id`.
- Explicit risk tier and approval requirement.
- Skill/plugin sources must be trusted and explicitly allowlisted.
- Filesystem/tool actions should be workspace-scoped by default.

## 4. Skill Contract Template (Required)
Use this structure for every skill definition.

```yaml
skill_id: SKL-<domain>-<name>
version: 1.0.0
status: draft
owner: <team_or_person>
description: <single-purpose function>

allowed_channels:
  - "#ctl-openclaw-commands"
  - "#prd-<project_id>-analytics"

required_role:
  - Owner
  - Operator

workflow_domain: <bookkeeping|product|research|operations>
risk_tier: <t1|t2|t3>
approval_required: <true|false>

input_schema:
  required_fields:
    - request_id
    - project_id
    - payload
  optional_fields:
    - context_summary

output_schema:
  required_fields:
    - request_id
    - status
    - result_summary
    - reference
    - next_action
  error_fields:
    - error_code
    - error_message

llm_policy:
  default_model_class: <A|B|C>
  max_model_class_without_approval: <A|B|C>
  max_input_tokens: <int>
  max_output_tokens: <int>
  retry_limit: 2
  fallback_chain:
    - <primary>
    - <secondary>
    - degraded_mode

execution_controls:
  timeout_seconds: <int>
  idempotency_key: request_id
  side_effects: <none|documented>
  rollback_strategy: <steps>

obsidian_persistence:
  path_template: "<vault_path>/<project_id>/"
  note_template: "<template_name>"
  required_metadata:
    - request_id
    - project_id
    - workflow_domain
    - timestamp_utc
    - reference

telemetry:
  required_log_fields:
    - request_id
    - project_id
    - workflow_domain
    - model_class
    - model_name
    - input_tokens
    - output_tokens
    - estimated_cost
    - latency_ms
    - status
```

## 5. Skill Readiness Checklist (Go/No-Go)
A skill is production-ready only when all are true:
- [ ] Contract fields are complete and reviewed.
- [ ] Allowed channels and role permissions are enforced.
- [ ] Input/output schema validation passes.
- [ ] LLM class, token limits, and budget guardrails are configured.
- [ ] Retry/fallback behavior is tested.
- [ ] Idempotency behavior is tested (safe re-run).
- [ ] Obsidian artifact writes succeed with required metadata.
- [ ] Telemetry fields appear in logs for every run.
- [ ] Rollback procedure is documented and tested.
- [ ] Dry-run and limited-rollout tests pass.
- [ ] Skill/plugin origin is trusted and allowlisted.
- [ ] Workspace-only filesystem constraints are verified where applicable.

## 6. Rollout Lifecycle (AOS-Specific)
1. **Draft:** define contract and risk tier.
2. **Validate:** run schema and policy checks.
3. **Dry Run:** execute without side effects.
4. **Shadow Run:** run in real traffic, do not take action.
5. **Limited Rollout:** enable for selected channels/users.
6. **Full Rollout:** enable broadly after KPI thresholds pass.
7. **Operate:** monitor cost, quality, and failure metrics.
8. **Retire/Replace:** deprecate with migration note and rollback path.

## 7. Example Skills for This AOS

### 7.1 Bookkeeping Reconciliation Skill
- Domain: bookkeeping
- Default model class: A
- Allowed channels: `#bk-<project_id>-reconcile`
- Output: reconciliation summary + discrepancy list + Obsidian note reference

### 7.2 Product Campaign KPI Digest Skill
- Domain: product
- Default model class: A/B
- Allowed channels: `#prd-<project_id>-analytics`
- Output: KPI delta digest + next campaign recommendation + Obsidian reference

### 7.3 Research Synthesis Skill
- Domain: research
- Default model class: B, escalates to C on complexity
- Allowed channels: `#res-topic-<topic>`, `#res-syntheses`
- Output: structured synthesis note + cited assumptions + next-stage proposal

## 8. Cost and Risk Guardrails
- Block execution when request/workflow/project budget caps are exceeded.
- Require approval for Class C usage when policy threshold is crossed.
- Require confirmation for high-risk side effects.
- Stop execution after retry limit and escalate to human review.

## 9. Skill KPIs
Track per skill version:
- Success rate
- First-attempt pass rate
- Median latency
- Cost per successful run
- Escalation rate (A/B -> C)
- Retry rate and failure mode distribution

## 10. Failure Handling Pattern
On failure:
1. Emit structured error payload.
2. Write incident/failed-run artifact to Obsidian.
3. Post actionable message to Discord with `request_id` and next steps.
4. If threshold exceeded, open incident workflow.

## 11. Anti-Patterns to Avoid
- Multi-purpose "do everything" skills.
- Missing schema validation.
- Hidden side effects without approvals.
- Unlimited retries or uncapped token usage.
- Writing outputs without reference IDs.
- Skills that cannot be re-run safely.

## 12. Related Documents
- [OpenClaw Agentic Operating System Build Procedure](OpenClaw-Agentic-Operating-System-Build-Procedure.md)
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md)
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md)
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md)
- [Master Index](Master-Index.md)

## 13. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.1.0 | Security Hardening v1: added trusted-source allowlist and workspace-scoped filesystem requirements for skills |
| 2026-02-23 | 1.0.0 | Initial AOS-customized skill specification template and readiness checklist |
