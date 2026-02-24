# OpenClaw Agentic Operating System Security Validation Runbook

## 1. Purpose
This runbook defines the recurring security validation steps for the OpenClaw Agentic Operating System.

## 2. Frequency and Ownership
- **Frequency:** Monthly (minimum), and after major config changes
- **Owner:** Security/Platform Owner
- **Operator:** Assigned Operator
- **Reviewer:** Assigned Reviewer

## 3. Preconditions
- Access to gateway host
- Access to Discord server settings
- Access to OpenClaw config and logs
- Access to Obsidian operational records

## 4. Validation Sequence
Execute in this order to ensure consistent evidence collection.

### Step 1: Gateway Exposure Check
- Verify gateway bind is loopback-only.
- Confirm no unintended public bind (**0.0.0.0**) is active.
- Validate remote access path is SSH tunnel or tailnet-based.
- Validate Ollama on local Windows host is reachable from VPS only via trusted tailnet path and is not publicly exposed.

**Pass Criteria**
- Gateway is loopback-bound and remote access is controlled.

**Fail Criteria**
- Any direct public exposure without approved exception.

### Step 2: DM Policy and Allowlist Check
- Verify DM policy is pairing mode by default.
- Validate unknown sender behavior (no command execution before approval).
- Verify allowlist entries are current and minimal.

**Pass Criteria**
- DM pairing flow works and allowlist scope is least-privilege.

### Step 3: Trust Boundary Check
- Confirm no mutually untrusted operator groups share one gateway trust boundary.
- Validate separation model (separate gateway/host/OS user) where required.

**Pass Criteria**
- Trust boundaries are documented and enforced.

### Step 4: Sandbox Policy Check
- Verify non-main/shared channels run under sandbox profile where required.
- Confirm high-risk tools are not broadly exposed in shared contexts.

**Pass Criteria**
- Sandbox rules match policy for shared/non-main sessions.

### Step 5: Security Audit Execution
- Run **openclaw security audit --deep**.
- Capture findings and classify severity.
- Confirm critical findings have immediate mitigation plan.

**Pass Criteria**
- No unresolved critical findings.

### Step 6: Secrets and Token Hygiene
- Confirm tokens/secrets are in **.env** or approved secret store only.
- Confirm no secrets are present in notes, docs, or source-controlled config.
- Validate token rotation readiness and last rotation date.

**Pass Criteria**
- Secret handling policy is fully compliant.

### Step 7: Logging and Evidence Integrity
- Confirm required telemetry fields exist (**request_id**, **project_id**, **workflow_domain**, status, timestamps, model/cost fields where applicable).
- Confirm incident and change records are persisted to Obsidian.

**Pass Criteria**
- Security-relevant logs are complete and traceable end-to-end.

## 5. Evidence to Capture
- Command output snippets (including security audit)
- Config screenshots or sanitized config excerpts
- Allowlist review record
- Sandbox policy verification record
- Remediation tickets with owners and due dates

## 6. Remediation SLA
- **Critical:** immediate containment + fix plan same day
- **High:** remediation within 7 days
- **Medium:** remediation within 30 days
- **Low:** backlog with documented rationale

## 7. Runbook Output Record Template
```markdown
# Security Validation Report - <YYYY-MM-DD>
- Run ID:
- Owner:
- Operator:
- Reviewer:
- Environment:

## Results by Step
1. Gateway Exposure Check: Pass/Fail
2. DM Policy and Allowlist Check: Pass/Fail
3. Trust Boundary Check: Pass/Fail
4. Sandbox Policy Check: Pass/Fail
5. Security Audit Execution: Pass/Fail
6. Secrets and Token Hygiene: Pass/Fail
7. Logging and Evidence Integrity: Pass/Fail

## Findings
- Critical:
- High:
- Medium:
- Low:

## Remediation Plan
- Item:
  - Owner:
  - Due Date:
  - Status:

## Final Decision
- Overall Status: Pass / Conditional Pass / Fail
- Next Review Date:
```

## 8. Go/No-Go Rule
- **Go:** all critical controls pass and no unresolved critical findings.
- **No-Go:** any unresolved critical finding or trust-boundary violation.

## 9. Related Documents
- [OpenClaw Agentic Operating System Build Procedure](OpenClaw-Agentic-Operating-System-Build-Procedure.md)
- [Discord Roles and Tasks](Discord-Roles-and-Tasks.md)
- [LLM Roles and Tasks](LLM-Roles-and-Tasks.md)
- [Obsidian Roles and Tasks](Obsidian-Roles-and-Tasks.md)
- [OpenClaw Skills Specification and Readiness (Customized for the OpenClaw Agentic Operating System)](OpenClaw-Skills-for-Agentic-Operating-System.md)
- [Master Index](Master-Index.md)

## 10. Revision History
| Date | Version | Change |
|---|---|---|
| 2026-02-24 | 1.1.0 | Added explicit Windows-host Ollama exposure validation to Step 1 for native-Windows inference architecture alignment |
| 2026-02-23 | 1.0.0 | Initial monthly security validation runbook for OpenClaw Agentic Operating System |
