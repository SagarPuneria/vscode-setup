---
mode: agent
description: 'Process IaC scan results, create structured remediation & tracking artifacts'
---

# Cycode IaC Remediation Plan Generator

This prompt runs AFTER an IaC scan (see `cycode-iac-scan.prompt.md`). It ingests the generated IaC scan artifacts, creates an organized remediation workspace, prioritizes misconfigurations, and establishes both human and machine-readable tracking for remediation progress.

## Prerequisites
- Completed IaC scan with outputs in: `ai-cycode-scans/<DATE>.<RUN_NUMBER>/`
- Source scan markdown report: `<TIME>_iac.md`
- Summary JSON: `<TIME>_iac_summary.json`
- (Optional) Raw scanner JSON (if produced by tooling)

## Objectives
1. Build a consistent remediation directory structure
2. Categorize & prioritize findings (Critical → Low)
3. Select top 20 per severity for focused remediation (or all, if < 20)
4. Provide per‑finding remediation playbooks (Terraform / Kubernetes / Docker / CloudFormation)
5. Generate master index (`index.md`) + machine index (`misconfigurations.json`)
6. Enable status transitions: pending → in_progress → completed / deferred / false_positive
7. Append reference section to original scan report linking to remediation artifacts

## Step 1: Locate Latest IaC Scan
1. List directories inside `ai-cycode-scans/` sorted descending (format: `YYYY-MM-DD.N`)
2. Pick most recent directory (or user-specified if variable provided)
3. Inside that directory, identify the newest file ending with `_iac_summary.json`
4. Derive base timestamp + run number

## Step 2: Parse Scan Summary
Extract from summary JSON:
- Total findings
- Severity breakdown (critical, high, medium, low)
- File type counts (terraform, kubernetes, docker, cloudformation, other)
- Compliance mapping (CIS / NIST / SOC2) if present
- Collect each finding object (fallback: parse markdown sections if raw JSON absent)

## Step 3: Build Remediation Directory Structure
Create (if not existing):
```
ai-cycode-scans/<DATE>.<RUN_NUMBER>/
├── <TIME>_iac.md
├── <TIME>_iac_summary.json
└── remediation-iac/
		├── index.md                      # Master tracking dashboard
		├── misconfigurations.json        # Machine-readable index
		├── compliance/                   # Compliance-focused summaries
		│   ├── CIS.md
		│   ├── NIST.md
		│   └── SOC2.md
		├── critical/
		│   ├── README.md
		│   ├── IAC-CRIT-001.md
		│   └── ...
		├── high/
		│   ├── README.md
		│   ├── IAC-HIGH-001.md
		│   └── ...
		├── medium/
		│   ├── README.md
		│   └── IAC-MED-001.md
		└── low/
				├── README.md
				└── IAC-LOW-001.md
```

## Step 4: Prioritization & Selection Logic
For each severity bucket:
1. Rank findings using weighted criteria:
	 - Severity weight (Critical=5, High=3, Medium=2, Low=1)
	 - Exposure potential (public network / privileged / broad IAM)
	 - Data sensitivity (touches PII / credentials / encryption resources)
	 - Blast radius (affects many workloads vs single resource)
2. Sort descending by composite risk score
3. Select top 20 (or fewer if less available)
4. All remaining (overflow) are still listed in `misconfigurations.json` but not given individual markdown remediation playbooks initially

## Step 5: Master Index (`remediation-iac/index.md`)
Template:
```markdown
# IaC Misconfiguration Remediation Index
**Scan Date:** [date time]
**Run:** [run number]
**Directory:** ./remediation-iac/

## Summary
| Severity | Total Found | Tracked (Top) | Completed | In Progress | Pending | Deferred | False Positives |
|----------|-------------|---------------|-----------|-------------|---------|----------|-----------------|
| Critical | C_total     | C_tracked     | 0         | 0           | C_tracked | 0        | 0               |
| High     | H_total     | H_tracked     | 0         | 0           | H_tracked | 0        | 0               |
| Medium   | M_total     | M_tracked     | 0         | 0           | M_tracked | 0        | 0               |
| Low      | L_total     | L_tracked     | 0         | 0           | L_tracked | 0        | 0               |

**Overall Progress:** 0 / [tracked_sum] (0%)

## Navigation
- [Critical](./critical/README.md)
- [High](./high/README.md)
- [Medium](./medium/README.md)
- [Low](./low/README.md)
- [Compliance (CIS)](./compliance/CIS.md) | [NIST](./compliance/NIST.md) | [SOC2](./compliance/SOC2.md)

## Critical Items
<!-- Auto-generated checklist -->
- [ ] IAC-CRIT-001: [Short title]
- [ ] IAC-CRIT-002: [Short title]

## Status Legend
Pending | In Progress | Completed | Deferred | False Positive

## SLA Guidance
- Critical: < 24h
- High: < 7 days
- Medium: < 30 days
- Low: < 90 days

## Last Updated
[timestamp]
```

## Step 6: Machine Index (`remediation-iac/misconfigurations.json`)
Schema example:
```json
{
	"scan_metadata": {
		"scan_date": "[ISO]",
		"run": "<DATE>.<RUN_NUMBER>",
		"source_report": "<TIME>_iac.md"
	},
	"statistics": {
		"severity": {"critical": C_total, "high": H_total, "medium": M_total, "low": L_total},
		"tracked": {"critical": C_tracked, "high": H_tracked, "medium": M_tracked, "low": L_tracked}
	},
	"findings": [
		{
			"id": "IAC-CRIT-001",
			"severity": "CRITICAL",
			"status": "pending",
			"resource_type": "aws_security_group",
			"file": "infrastructure/sg.tf",
			"line": 42,
			"description": "Security group allows 0.0.0.0/0 on port 22",
			"frameworks": ["CIS"],
			"controls": ["CIS-4.1"],
			"risk_factors": ["public_ingress", "ssh_exposed"],
			"recommended_fix": "Restrict CIDR or remove rule",
			"created": "[ISO]",
			"updated": "[ISO]",
			"assigned_to": null,
			"effort": "30m",
			"history": [
				{"date": "[ISO]", "status": "pending", "note": "Imported from scan"}
			]
		}
	]
}
```

## Step 7: Severity README Templates
Example (`critical/README.md`):
```markdown
# Critical Misconfigurations (Top 20)
**Total Critical Found:** [C_total] (Tracking top [C_tracked])

| ID | Resource | File | Status | Assigned | Effort |
|----|----------|------|--------|----------|--------|
| [IAC-CRIT-001](./IAC-CRIT-001.md) | aws_security_group | infrastructure/sg.tf | Pending | - | 30m |

## Progress
- Completed: 0/[C_tracked]
- In Progress: 0/[C_tracked]
- Pending: [C_tracked]/[C_tracked]

## Notes
Patterns: e.g., Excessive open ingress rules.
```

Repeat similar format for High / Medium / Low.

## Step 8: Individual Misconfiguration File Template
Filename: `IAC-<SEVERITY>-NNN.md`
```markdown
# IAC-<SEVERITY>-NNN: [Short Title]

## Status
**Current:** Pending  
**Severity:** <SEVERITY>  
**Assigned To:** Unassigned  
**Effort Estimate:** [e.g., 30m]

## Finding Details
- **Resource Type:** [terraform/kubernetes/docker/cloudformation + specific]
- **File:** `path/to/file`
- **Line:** N
- **Description:** [Full text]
- **Frameworks/Controls:** CIS:4.1, NIST:AC-xx
- **Risk Factors:** [list]

## Risk Assessment
- **Exposure:** [Internal/Public]
- **Impact:** [What compromise enables]
- **Likelihood:** [High/Med/Low]
- **Overall Risk:** [Derived]

## Code Context
```hcl|yaml|dockerfile
[3-5 lines before]
>> LINE N: [Issue (masked if sensitive)]
[3-5 lines after]
```

## Remediation Guidance
### Terraform
- Principle: Least privilege / restrict ingress / enable encryption
- Example Fix:
```hcl
resource "aws_security_group" "example" {
	ingress {
		from_port   = 22
		to_port     = 22
		protocol    = "tcp"
		cidr_blocks = ["10.0.0.0/24"]  # Restrict to corporate network
	}
}
```

### Kubernetes (example pattern)
```yaml
# Before: privileged true, no limits
securityContext:
	privileged: true

# After: drop privileges
securityContext:
	runAsNonRoot: true
	allowPrivilegeEscalation: false
	capabilities:
		drop: ["ALL"]
```

### Docker (example pattern)
```dockerfile
# Before: no explicit user
FROM python:3.11

# After: add non-root user
FROM python:3.11
RUN useradd -m appuser
USER appuser
```

## Step-by-Step Actions
1. Reproduce finding locally (lint / terraform validate / kubectl apply --dry-run)
2. Apply configuration changes
3. Run validation (terraform plan / kubectl diff / docker build)
4. Re-run IaC scan focused on file
5. Open PR referencing this ID
6. Update status to Completed upon merge

## Verification Checklist
- [ ] Static validation passes
- [ ] No new high-severity drifts introduced
- [ ] Scan re-run clean for this ID
- [ ] Peer review completed
- [ ] Linked ticket closed

## Status History
| Date | Status | By | Notes |
|------|--------|----|-------|
| [date] | Pending | System | Imported from scan |

## Notes
[Additional context]

---
**Created:** [timestamp]  
**Last Updated:** [timestamp]  
**Original Scan:** [link relative path]
```

## Step 9: Compliance Summaries
Generate one file per framework summarizing mapped control failures:
`compliance/CIS.md` example:
```markdown
# CIS Benchmark Mapping
| Control | Count | Representative IDs |
|---------|-------|--------------------|
| CIS-4.1 | 3     | IAC-CRIT-001, IAC-HIGH-003, IAC-MED-010 |

## Highest Priority Controls
1. CIS-4.1: Restrict ingress sources
2. CIS-2.2: Enable encryption at rest
```

## Step 10: Append Note to Original Scan Report
Append at end of `<TIME>_iac.md`:
```markdown
---
## Remediation Plan Generated
Artifacts created in: `./remediation-iac/`
- Master Index: remediation-iac/index.md
- Machine Index: remediation-iac/misconfigurations.json
- Per Severity READMEs and individual issue files

Start remediation with Critical items: [link](./remediation-iac/critical/README.md)
```

## Step 11: Final Summary Output (Console / Chat)
```
✅ IaC remediation plan created
• Total findings: [N] (Critical: C | High: H | Medium: M | Low: L)
• Tracked (top per severity): [tracked_sum]
• Location: ai-cycode-scans/<DATE>.<RUN_NUMBER>/remediation-iac/

Next: Open remediation-iac/index.md and begin with IAC-CRIT-001
```

## Status Workflow Rules
- Allowed transitions: pending → in_progress → completed | deferred | false_positive
- `deferred` requires justification & review date
- `false_positive` requires rationale & reviewer sign-off

## Error Handling
- Missing scan files: Abort with instruction to run IaC scan
- No findings: Create `remediation-iac/` with index noting "No misconfigurations detected"
- Fewer than 20 per severity: Track actual count
- Write failures: Emit path + permission hint

## Automation Hooks (Optional/Future)
- CI job can parse `misconfigurations.json` to fail build if new Critical present
- Badge generation from progress stats
- Auto-update status on merge referencing ID in commit messages (`fix: IAC-CRIT-003`)

## Success Criteria
- Directory structure fully generated
- Master and machine indexes present & valid schema
- Individual markdown files for tracked findings exist
- Compliance summaries generated when data available
- Original report appended with plan reference

## Maintenance Guidelines
To update a finding status:
1. Edit corresponding `IAC-<SEVERITY>-NNN.md`
2. Update checklist & status in `index.md`
3. Reflect change in `misconfigurations.json` (append history entry)
4. If completed, ensure verification checklist fully checked

## Notes
- Never commit sensitive values (keys, account IDs beyond masking policy)
- Prefer least privilege and deny-by-default patterns
- Encourage small, atomic PRs per remediation item
