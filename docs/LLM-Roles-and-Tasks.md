# LLM Roles and Tasks in the OpenClaw Architecture

## 1. Purpose
This document defines how LLMs are selected, routed, governed, and cost-controlled in the OpenClaw system.

> **Plain-English Note:** Different LLMs should have different jobs. Cheap/fast models do routine work, and high-capability models are used only when truly needed.

## 2. Scope
This document covers:
- LLM role classes and workload assignment
- Channel-based routing policy
- Cost controls and budget guardrails
- Escalation/fallback logic
- Monitoring, KPIs, and weekly governance

This document does not cover:
- Hardware deployment specifics for Ollama hosts
- Vendor billing setup details

### 2.1 Cross-Document Orchestration
This LLM guide is part of a coordinated operating set:
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md) defines channel topology, command controls, and workflow intake.
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md) defines structured persistence, metadata contracts, and recovery records.

Execution chain in practice:
1. Discord channel and policy classify the request.
2. LLM routing/cost policy assigns model class and limits.
3. Results, evidence, and decision metadata are written to Obsidian.
4. Discord receives completion status and references.

> **Plain-English Note:** LLM policy decides how the system thinks; Discord decides where work starts; Obsidian preserves what happened.

### 2.2 Security Alignment (OpenClaw Trust Model)
- Treat authenticated gateway callers as trusted operators within one trust boundary.
- Do not use one gateway for mutually untrusted operator groups.
- Prompt injection risk is expected and should be handled by policy/validation, not assumed as a sandbox boundary.
- Workspace memory and plugin code are trusted local state; do not treat them as isolation boundaries.

## 3. LLM Role Model

### 3.1 Role Classes
- **Class A - Fast/Low-Cost:** extraction, tagging, classification, short summaries.
- **Class B - Balanced:** drafting, rewriting, medium-complexity reasoning.
- **Class C - High-Capability:** deep synthesis, strategy, decision-critical analysis.

### 3.2 Role Assignment Principles
- Default to the lowest-cost model class that can meet quality requirements.
- Escalate to a higher class only on defined triggers.
- Keep role boundaries explicit to prevent unnecessary spend.

## 4. Routing Policy by Workflow Domain

### 4.1 Bookkeeping and Reconciliation
- Default: **Class A**
- Escalate to Class B only for anomaly explanations requiring reasoning depth.
- Avoid Class C unless explicitly approved.

### 4.2 Product Marketing and Sales Channels
- Drafting and transformation: **Class A/B**
- Strategic positioning or complex synthesis: **Class C** with approval gate.

### 4.3 Research and Idea Evolution
- Source extraction and preprocessing: **Class A**
- Synthesis and recommendation: **Class C**
- Final executive summary may use **Class B/C** based on complexity.

### 4.4 Operations and Incident Support
- Triage and structured logging: **Class A**
- Root-cause narrative and multi-factor analysis: **Class B/C**

## 5. Escalation and Fallback Logic

### 5.1 Escalation Triggers
Escalate model class only when one or more conditions are true:
- Output fails quality checks (format, completeness, factual consistency checks).
- Task complexity exceeds policy threshold.
- Prior attempt returns low confidence or unresolved ambiguity.

### 5.2 Fallback Chain
- Primary class attempt -> secondary class attempt -> degraded response mode.
- In degraded mode, bot returns a safe partial result + next action.

### 5.3 Retry Limits
- Maximum 2 retries per model class.
- Use exponential backoff to avoid runaway token spend.
- Stop and require human review when retries are exhausted.

## 6. Cost Control Policy v1

### 6.1 Budget Levels
Set budgets at three levels:
- **Per Request:** token ceilings for input/output.
- **Per Workflow/Channel:** daily token and spend caps.
- **Per Project:** weekly and monthly spend caps.

### 6.2 Hard Guardrails
- Block execution if budget cap is exceeded.
- Require approval for Class C usage above threshold.
- Require approval for large-context requests above token limit.
- Enforce maximum output length for routine tasks.

### 6.3 Context Cost Hygiene
- Use summarized context, not full raw channel history.
- Keep reusable context blocks cached and versioned.
- Deduplicate repeated source content before prompting.
- Prefer structured prompts and structured outputs.

### 6.4 Waste Reduction Controls
- Batch similar low-risk tasks where practical.
- Avoid re-running unchanged tasks.
- Attach idempotency keys to long-running jobs.
- Cancel stale requests that exceed business value threshold.

## 7. Quality Gates Before High-Cost Calls
Before Class C execution, verify:
- Task cannot be completed by Class A/B.
- Expected value justifies higher token spend.
- Required inputs are complete and scoped.
- Project budget remaining is sufficient.

## 8. Logging and FinOps Telemetry
Every LLM execution record should include:
- **request_id**
- **project_id**
- **workflow_domain**
- **model_class**
- **model_name**
- **input_tokens**
- **output_tokens**
- **estimated_cost**
- **latency_ms**
- **status**
- **escalated_from** (if any)

> **Plain-English Note:** If you do not log costs and outcomes together, you cannot optimize spend effectively.

## 9. KPI Dashboard (Weekly Review)
Track and review:
- Cost per completed workflow
- Cost per project per week
- Escalation rate (A/B -> C)
- Token waste rate (failed or aborted runs)
- Median latency by workflow domain
- Success rate on first attempt

## 10. Governance Cadence
- **Daily:** budget threshold alerts and anomaly checks.
- **Weekly:** top-cost workflows, escalation analysis, optimization actions.
- **Monthly:** policy retuning (caps, routing, quality thresholds) and **openclaw security audit --deep** review.
- **Quarterly:** strategic vendor/model portfolio review.

## 11. LLM Incident Triggers
Open an LLM incident when:
- Project budget burn exceeds threshold unexpectedly.
- Escalation rate spikes above baseline.
- Failure rate or timeout rate exceeds SLA targets.
- Model outputs violate required format/security constraints repeatedly.

## 12. Implementation Checklist
- [ ] Define model classes and approved models.
- [ ] Map each Discord channel family to default model class.
- [ ] Configure escalation and fallback policy.
- [ ] Set per-request/workflow/project budget caps.
- [ ] Enable FinOps telemetry fields in execution logs.
- [ ] Create weekly KPI review routine.
- [ ] Test degraded mode behavior.
- [ ] Confirm trust-boundary design (no mutually untrusted operators on one gateway).
- [ ] Include monthly security-audit findings in governance review.

## 13. Starter Policy Matrix
| Workflow Domain | Default Class | Max Class Without Approval | Escalation Allowed | Budget Strictness |
|---|---|---|---|---|
| Bookkeeping | A | B | Yes | High |
| Product Analytics | A | B | Yes | High |
| Product Marketing Drafts | B | C | Yes | Medium |
| Research Synthesis | B | C | Yes | Medium |
| Incident RCA Narrative | B | C | Yes | Medium |
| Destructive/High-Risk Ops Advice | B | C (approval required) | Controlled | Very High |

## 14. Plain-English Quick Rules
- Start cheap.
- Escalate only when needed.
- Stop when budget says stop.
- Log every run.
- Review weekly and tune.

## 15. Internet Research Policy v1
This policy governs when and how LLM workflows can access internet sources.

### 15.1 Research Access Model
Internet access must be tool-mediated and policy-scoped.

Access tiers:
- **Tier R1 (Default):** search + fetch metadata/snippets only.
- **Tier R2 (Controlled):** full-page retrieval and extraction for approved channels.
- **Tier R3 (Restricted):** broad autonomous crawling/action is disabled unless explicitly approved.

### 15.2 Channel and Domain Controls
- Internet-enabled workflows must be restricted to approved research channels.
- Apply allowlist-first domain policy for critical workflows.
- Block high-risk or irrelevant source classes unless explicitly required.
- Log all fetched source domains for audit review.

### 15.3 Research Execution Pattern
Use staged retrieval to improve quality and control costs:
1. Broad discovery (search pass)
2. Source triage (relevance and trust check)
3. Targeted deep read (top sources)
4. Synthesis with explicit citations
5. Contradiction check and confidence summary

### 15.4 Context Window and Token Controls
- Keep live context usage below ~80% of model context window.
- Prefer carrying forward structured summaries instead of raw page dumps.
- Enforce per-stage token budgets (discover/triage/read/synthesize).
- Trigger compaction/summarization before exceeding threshold.

### 15.5 Evidence and Citation Requirements
- No unsupported claims: all material claims must reference retrieved evidence.
- Output must include sources, extraction timestamp, and confidence note.
- When evidence is insufficient, output must state uncertainty and next-step research.

### 15.6 Safety and Approval Gates
- High-cost deep research runs require budget and policy checks before execution.
- High-risk outputs (strategic or operationally sensitive) require reviewer sign-off.
- If policy constraints block retrieval, system returns degraded safe response and escalation path.

### 15.7 Telemetry Requirements (Research Runs)
Research runs must log:
- **request_id**
- **project_id**
- **workflow_domain**
- **model_class** / **model_name**
- **context_window_percent**
- **input_tokens** / **output_tokens** / **estimated_cost**
- **sources_fetched_count**
- **source_domains**
- **status**

### 15.8 Operational Checklist (Internet Research)
- [ ] Research channels requiring internet access are explicitly allowlisted.
- [ ] Tier model (R1/R2/R3) is assigned per workflow.
- [ ] Context threshold and auto-compaction controls are configured.
- [ ] Citation and confidence output requirements are enforced.
- [ ] Source domain logs are retained for audit.
- [ ] Budget guardrails are tested for deep research mode.

## 16. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-23 | 1.2.0 | Added Internet Research Policy v1 with access tiers, context controls, citation requirements, and research telemetry |
| 2026-02-23 | 1.1.0 | Security Hardening v1: added trust-model alignment and monthly security audit governance requirements |
| 2026-02-23 | 1.0.1 | Added cross-document orchestration links and unified execution-chain guidance with Discord and Obsidian docs |
| 2026-02-23 | 1.0.0 | Initial LLM roles, routing, and cost-control governance document |
