---
mode: agent
description: 'Process secrets scan results, create remediation plan with tracking'
---

# Cycode Secrets Remediation Plan Generator

This prompt executes AFTER a secrets scan has been completed. It processes scan results, creates an organized remediation plan, and establishes a tracking system for vulnerabilities.

## Prerequisites
- Secrets scan must be completed (using cycode-secrets-scan.prompt.md)
- Scan results available in `ai-cycode-scans/[date].[run]/` directory
- JSON summary file present

## Objectives
1. Create standardized remediation workspace paralleling IaC plan conventions
2. Prioritize secrets by severity & contextual risk (scope, age, blast radius)
3. Generate human dashboard (`index.md`) and machine index (`vulnerabilities.json`)
4. Provide per‑secret remediation playbooks with rotation + verification steps
5. Enable auditable status transitions & historical tracking
6. Support automation hooks for CI quality gates & progress reporting

## Task: Create Remediation Plan and Tracking System

### Step 1: Locate Most Recent Scan Results
1. Navigate to `ai-cycode-scans/` directory
2. Find the most recent scan directory (format: `YYYY-MM-DD.N`)
3. Load the secrets scan JSON summary
4. Load the detailed markdown report

### Step 2: Create Remediation Directory Structure
Create the following structure within the scan directory:

```
ai-cycode-scans/[date].[run]/
├── [time]_secrets.md                    # Original scan report
├── [time]_secrets_summary.json          # Original summary
├── remediation/
│   ├── index.md                         # Master tracking index
│   ├── vulnerabilities.json             # Machine-readable index
│   ├── critical/
│   │   ├── README.md                    # Top 20 critical issues
│   │   ├── VULN-CRIT-001.md
│   │   ├── VULN-CRIT-002.md
│   │   └── ...
│   ├── high/
│   │   ├── README.md                    # Top 20 high severity issues
│   │   ├── VULN-HIGH-001.md
│   │   ├── VULN-HIGH-002.md
│   │   └── ...
│   ├── medium/
│   │   ├── README.md                    # Top 20 medium severity issues
│   │   ├── VULN-MED-001.md
│   │   └── ...
│   └── low/
│       ├── README.md                    # Top 20 low severity issues
│       ├── VULN-LOW-001.md
│       └── ...
```

### Step 3: Parse and Categorize Vulnerabilities
1. Extract all findings from scan results
2. Group by severity: CRITICAL, HIGH, MEDIUM, LOW
3. Sort each group by:
   - Risk score (if available)
   - File criticality (production > development > test)
   - Exposure age (older = higher priority)
4. Select top 20 from each severity category

### Step 4: Generate Master Index (`remediation/index.md`)

```markdown
# Secrets Remediation Master Index
**Scan Date:** [date and time]
**Branch:** [branch name]
**Total Vulnerabilities:** [count]

## Summary Statistics
| Severity | Total Found | Tracked (Top) | Completed | In Progress | Pending | Deferred | False Positives |
|----------|-------------|---------------|-----------|-------------|---------|----------|-----------------|
| Critical | [C_total]   | [C_tracked]   | 0         | 0           | [C_tracked] | 0        | 0               |
| High     | [H_total]   | [H_tracked]   | 0         | 0           | [H_tracked] | 0        | 0               |
| Medium   | [M_total]   | [M_tracked]   | 0         | 0           | [M_tracked] | 0        | 0               |
| Low      | [L_total]   | [L_tracked]   | 0         | 0           | [L_tracked] | 0        | 0               |

**Overall Progress:** 0 / ([C_tracked]+[H_tracked]+[M_tracked]+[L_tracked]) = 0%

## Quick Navigation
- [Critical Issues](./critical/README.md) - **IMMEDIATE ACTION REQUIRED**
- [High Issues](./high/README.md) - Address within 7 days
- [Medium Issues](./medium/README.md) - Address within 30 days
- [Low Issues](./low/README.md) - Address within 90 days

## Completion Tracking
**Overall Progress:** 0/[total] (0%)

### Critical Issues
- [ ] VULN-CRIT-001: [Brief description] - **Status:** Pending
- [ ] VULN-CRIT-002: [Brief description] - **Status:** Pending
[... continue for all critical ...]

### High Issues
- [ ] VULN-HIGH-001: [Brief description] - **Status:** Pending
[... continue for all high ...]

### Medium Issues
- [ ] VULN-MED-001: [Brief description] - **Status:** Pending
[... continue for top 20 ...]

### Low Issues
- [ ] VULN-LOW-001: [Brief description] - **Status:** Pending
[... continue for top 20 ...]

## Status Legend
- **Pending:** Not started
- **In Progress:** Remediation underway
- **Completed:** Fixed and verified
- **Deferred:** Documented reason for deferral
- **False Positive:** Marked as non-issue with justification

## Last Updated
[Auto-generated timestamp]
```

### Step 5: Generate vulnerabilities.json Index

Create `remediation/vulnerabilities.json`:

```json
{
  "scan_metadata": {
    "scan_date": "[ISO timestamp]",
    "branch": "[branch]",
    "scan_directory": "[directory path]",
    "total_findings": N
  },
  "statistics": {
    "critical": {"total": N, "tracked": N, "completed": 0, "in_progress": 0, "pending": N},
    "high": {"total": N, "tracked": N, "completed": 0, "in_progress": 0, "pending": N},
    "medium": {"total": N, "tracked": N, "completed": 0, "in_progress": 0, "pending": N},
    "low": {"total": N, "tracked": N, "completed": 0, "in_progress": 0, "pending": N}
  },
  "vulnerabilities": [
    {
      "id": "VULN-CRIT-001",
      "severity": "CRITICAL",
      "status": "pending",
      "secret_type": "[e.g., AWS Access Key]",
      "file": "[file path]",
      "line": N,
      "detection_rule": "[rule name]",
      "created_date": "[ISO timestamp]",
      "updated_date": "[ISO timestamp]",
      "assigned_to": null,
      "estimated_effort": "[e.g., 2h]",
      "completion_date": null,
      "verification_status": null,
      "notes": []
    }
  ]
}
```

### Step 6: Create Severity Category READMEs

For each severity directory (critical/, high/, medium/, low/), create a README.md:

```markdown
# [SEVERITY] Priority Secrets - Top 20
**Scan Date:** [date]
**Category:** [SEVERITY]
**Total in Category:** [N] (showing top 20)

## ⚠️ Urgency Level
**[CRITICAL: IMMEDIATE | HIGH: 7 days | MEDIUM: 30 days | LOW: 90 days]**

## Issues Overview
| ID | File | Secret Type | Status | Assigned | Est. Effort |
|----|------|-------------|--------|----------|-------------|
| [VULN-XXX-001](./VULN-XXX-001.md) | [file] | [type] | Pending | - | 2h |
| [VULN-XXX-002](./VULN-XXX-002.md) | [file] | [type] | Pending | - | 1h |
[... continue for all 20 ...]

## Progress Summary
- **Completed:** 0/20 (0%)
- **In Progress:** 0/20 (0%)
- **Pending:** 20/20 (100%)

## Notes
[Any category-specific notes or patterns observed]
```

### Step 7: Create Individual Vulnerability Files

For each vulnerability (top 20 per category), create `VULN-[SEVERITY]-[NNN].md`:

```markdown
# VULN-[SEVERITY]-[NNN]: [Secret Type] in [filename]

## Status
**Current:** Pending  
**Assigned To:** Unassigned  
**Priority:** [SEVERITY]  
**Estimated Effort:** [e.g., 2h]

## Vulnerability Details
- **Secret Type:** [e.g., AWS Access Key ID]
- **File:** `[full/path/to/file.ext]`
- **Line Number:** [N]
- **Detection Rule:** [rule name]
- **First Detected:** [date]

## Risk Assessment
- **Exposure Level:** [Public Repo / Private Repo / Local Only]
- **Access Scope:** [What this secret can access]
- **Potential Impact:** [Describe potential damage]
- **Exploit Difficulty:** [Easy / Medium / Hard]

## Code Context
```[language]
[Show 3-5 lines before]
>>> LINE [N]: [THE LINE WITH SECRET - MASKED]
[Show 3-5 lines after]
```

## Remediation Steps

### 1. Immediate Actions (Do First)
- [ ] **Rotate/Revoke the exposed credential immediately**
  - Service: [AWS/GitHub/etc.]
  - Action: [Specific steps to rotate]
  - Documentation: [Link to service's credential rotation guide]

### 2. Remove from Code
- [ ] Remove the hardcoded secret from line [N]
- [ ] Verify the secret doesn't appear elsewhere in the file
- [ ] Search codebase for other instances: `git grep -n "[partial_secret]"`

### 3. Implement Secure Storage
- [ ] Store credential in: [Vault/AWS Secrets Manager/Azure Key Vault/Environment Variable]
- [ ] Update code to retrieve from secure storage
- [ ] Add configuration documentation

### 4. Update Git History (If Needed)
- [ ] Determine if secret exists in git history: `git log -S "[partial_secret]" --all`
- [ ] If in history, document whether rewrite is needed
- [ ] Follow team's git history rewrite policy

### 5. Verification
- [ ] Re-run secrets scan on modified file
- [ ] Confirm no secrets detected
- [ ] Test application functionality with new secret retrieval method
- [ ] Update this file status to "Completed"

## Code Change Example

**Before:**
```[language]
api_key = "AKIAIOSFODNN7EXAMPLE"  # WRONG - Hardcoded
```

**After:**
```[language]
import os
api_key = os.environ.get('AWS_API_KEY')  # Correct - From environment
# Or use AWS Secrets Manager, Vault, etc.
```

## Verification Checklist
- [ ] Secret rotated in source system
- [ ] Secret removed from code
- [ ] Secure retrieval mechanism implemented
- [ ] Re-scan shows no detection
- [ ] Application tested and working
- [ ] Documentation updated
- [ ] PR reviewed and approved

## Status History
| Date | Status | Updated By | Notes |
|------|--------|------------|-------|
| [date] | Pending | System | Created from scan results |

## Notes
[Space for additional context, blockers, or decisions]

---
**Created:** [timestamp]  
**Last Updated:** [timestamp]  
**Linked Scan:** [link to original scan report]
```

### Step 8: Generate Summary Output

Create a final summary file: `remediation/REMEDIATION_PLAN_SUMMARY.md`

```markdown
# Secrets Remediation Plan - Executive Summary

## Plan Created
**Date:** [timestamp]
**Scan Reference:** [scan directory]
**Total Vulnerabilities:** [N]

## Remediation Scope
This plan addresses the **top 20 vulnerabilities** from each severity category:
- **Critical:** [N] of [total] ([percentage]%)
- **High:** [N] of [total] ([percentage]%)
- **Medium:** [N] of [total] ([percentage]%)
- **Low:** [N] of [total] ([percentage]%)

**Total Issues Tracked:** [sum of top 20s]

## Directory Structure
All remediation files are located in:
```
[full path to remediation directory]
```

## Key Files
1. **Master Index:** `remediation/index.md` - Your central tracking dashboard
2. **Machine Index:** `remediation/vulnerabilities.json` - For automation/tooling
3. **Category READMEs:** Quick overview of each severity level
4. **Individual Tasks:** Detailed remediation steps for each vulnerability

## Getting Started
1. Start with CRITICAL issues: `cd remediation/critical/`
2. Review the README.md for overview
3. Pick an issue: Open `VULN-CRIT-001.md`
4. Follow the remediation steps
5. Update status in `index.md` and `vulnerabilities.json` as you progress

## Recommended Workflow
1. **Daily:** Address 2-3 CRITICAL issues
2. **Weekly:** Review and update index.md with progress
3. **Bi-weekly:** Re-run scan to catch new issues
4. **Monthly:** Review and prioritize MEDIUM/LOW issues

## Automation Opportunities
- Use `vulnerabilities.json` for CI/CD integration
- Script progress reports from JSON
- Auto-update status based on git commits

## Next Steps
- [ ] Review critical issues
- [ ] Assign owners to top 5 critical vulnerabilities
- [ ] Schedule daily standup item for remediation progress
- [ ] Set up alerts for new secret detections

---
**Generated by:** Cycode Secrets Remediation Plan Generator
**Questions?** See documentation in `.github/prompts/`
```

## Step 9: Update Original Scan Report

Append to the original `[time]_secrets.md` report:

```markdown

---

## Remediation Plan Generated
**Date:** [timestamp]
**Location:** `./remediation/`

A comprehensive remediation plan has been created with:
- Top 20 vulnerabilities per severity level
- Individual task files with detailed steps
- Master tracking index for progress monitoring
- Machine-readable JSON index for automation

**Start here:** [View Master Index](./remediation/index.md)
```

## Success Criteria
- ✅ All directory structure created correctly
- ✅ Master index.md generated with all vulnerabilities listed
- ✅ vulnerabilities.json created with proper schema
- ✅ Top 20 selected from each severity category
- ✅ Individual vulnerability files created with detailed steps
- ✅ Category READMEs generated
- ✅ Summary report created
- ✅ Original scan report updated with link to remediation plan

## Output to User
Display:
```
✅ Remediation plan created successfully!

📊 Summary:
   - Total vulnerabilities found: [N]
   - Tracked for remediation: [M]
   - Critical: [C] | High: [H] | Medium: [M] | Low: [L]

📁 Location: ai-cycode-scans/[date].[run]/remediation/

🚀 Quick Start:
   1. Open: remediation/index.md
   2. Start with: remediation/critical/README.md
   3. Track progress in index.md

⚡ Priority Action:
   Begin with VULN-CRIT-001 - [brief description]
```

## Error Handling
- If scan results not found: Prompt to run secrets scan first
- If no vulnerabilities: Create empty structure with "No issues found" message
- If directory creation fails: Report error with permissions check
- If fewer than 20 in category: Create files for actual count

## Maintenance Instructions
When updating status of a vulnerability:
1. Update the individual VULN file's status section
2. Update the checkbox in index.md
3. Update the status in vulnerabilities.json
4. Update the statistics table in index.md
5. Add entry to Status History table in VULN file

## Status Workflow Rules
- Allowed transitions: `pending → in_progress → completed` OR `pending → deferred` OR `pending → false_positive`
- Moving to `deferred` requires: justification + re-review date (add to Notes + machine index)
- Marking `false_positive` requires: rationale + reviewer (security/lead) acknowledgement
- `completed` requires all verification checklist boxes checked
- History entry appended on every status change with timestamp & actor

## Automation Hooks
- CI can parse `vulnerabilities.json` and fail build if new Critical remains `pending`
- Nightly job can compute remediation velocity (closed per day) from history entries
- Commit messages referencing IDs (e.g., `sec: remove hardcoded token VULN-CRIT-004`) can auto-update status to `in_progress` or `completed` post-merge
- Dashboard integration: convert JSON into badge (e.g., Critical Remaining)

## Alignment With IaC Plan
| Aspect | Secrets Plan | IaC Plan Parallel |
|--------|--------------|--------------------|
| Directory | `remediation/` | `remediation-iac/` |
| Machine Index | `vulnerabilities.json` | `misconfigurations.json` |
| Item IDs | `VULN-<SEVERITY>-NNN` | `IAC-<SEVERITY>-NNN` |
| Top Selection | Top 20 / severity | Top 20 / severity |
| Statuses | pending, in_progress, completed, deferred, false_positive | Same |
| History | `Status History` table + JSON notes | Same (history array) |

If future consolidation desired, both could adopt a generic naming scheme (`findings.json`, `remediation-secrets/`, `remediation-iac/`). Current form preserves backward compatibility.

## Automation Extension Ideas
- Add optional `rotated` boolean once credential rotation confirmed
- Add `rotation_ticket` field linking to external ITSM / cloud console request
- Add `exposure_window_days` computed (first_detected → rotation date)

## Data Integrity Checks
Before marking plan generation successful:
1. Validate JSON schema fields presence: id, severity, status, file, line
2. Ensure all tracked IDs appear in `index.md` checklists
3. Confirm no duplicate IDs across severity folders
4. Count mismatch (index vs JSON stats) aborts with corrective instruction

## Deprecation / Evolution Notes
Future iteration may:
- Introduce `severity_weight` for dynamic prioritization
- Consolidate secrets + IaC + SAST + SCA into unified risk dashboard
- Provide delta mode (only new findings since last scan)

## Security Guardrails
- Never write actual secret value or even full hash; partial last 4–6 chars only when needed for identification
- Avoid storing rotation metadata that includes sensitive console URLs or unredacted ARNs if policy disallows

## Verification Quick Script (Optional Concept)
An automation could iterate `vulnerabilities.json` and re-scan only affected files to validate `completed` items remain clean (not implemented here, left for tooling layer).

## Notes
- NEVER expose actual secret values in any file
- All examples should use masked/example values
- Prioritize based on actual risk, not just severity label
- Link related vulnerabilities (e.g., multiple secrets in same file)