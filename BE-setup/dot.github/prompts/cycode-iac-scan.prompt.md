---
mode: 'agent'
description: 'Scan Infrastructure as Code files for security misconfigurations and compliance violations'
---

# Cycode IaC Security Scan with MCP Tools

Execute Infrastructure as Code security analysis using Cycode MCP tools.

## Context Required
- IaC files (Terraform, Kubernetes, Docker, CloudFormation)
- Changed configuration files
- Compliance requirements

## Task: Analyze Infrastructure Security

### Step 1: Collect Uncommitted IaC Files (Working Tree Only)
Gather ONLY IaC files currently modified (unstaged/staged but not committed) or newly added.

1. Uncommitted tracked changes: `git diff --name-only HEAD`
2. Untracked new files: `git ls-files --others --exclude-standard`
3. Combine, de-duplicate list.
4. Filter by IaC patterns:
  - Terraform: `*.tf`, `*.tfvars`
  - Kubernetes: `*.yaml`, `*.yml` (must contain both `apiVersion:` and `kind:`)
  - Docker: `Dockerfile`, `docker-compose*.yml`
  - CloudFormation: `*.cfn.json`, `*.cfn.yaml`
5. Exclude vendor / build / cache dirs: `node_modules/`, `.terraform/`, `dist/`, `__pycache__/`.
6. If set is empty:
  - Output: `No uncommitted IaC changes detected. Scan skipped.`
  - Stop unless `FULL_SCAN=1` override.
7. If `FULL_SCAN=1` set: include all matching IaC files (excluding ignored dirs) and mark `(full scan override)`.
8. Provide count summary before scan.

### Step 2: Execute MCP IaC Scan (Working Tree Set)
Use the Cycode MCP tool `cycode_scan_iac` with parameters:
- `scan_type`: "iac"
- `paths`: [changed IaC files list]
- `compliance_frameworks`: ["CIS", "NIST", "SOC2"]
- `severity_threshold`: "ALL"

Notes:
- Do not include files outside working tree changes unless FULL_SCAN override used.
- If `FULL_SCAN=1` override: include all IaC files and mark summary `(full scan override)`.
- If >300 uncommitted IaC files, batch (e.g., 150) and merge results.

### Step 3: Create Report Directory
```bash
SCAN_DATE=$(date +%Y-%m-%d)
SCAN_TIME=$(date +%H_%M)
```
1. Use existing run number or create new
2. Create report path: `ai-cycode-scans/${SCAN_DATE}.${RUN_NUMBER}/`

### Step 4: Generate Structured Report
Create: `${SCAN_TIME}_iac.md`

Structure:
```markdown
# Cycode IaC Scan Report
**Date:** [timestamp]
**Scan Type:** Infrastructure as Code
**Run:** [run number]

## Executive Summary
- IaC files scanned: [count by type]
- Critical misconfigurations: [count]
- Compliance violations: [count]

## Findings by File Type
### Terraform
[List findings]

### Kubernetes
[List findings]

### Docker
[List findings]
```

### Step 5: Generate Remediation Tasks
For each misconfiguration, create task:

```markdown
### Task [N]: Fix [issue] in [file]
**Priority:** [severity]
**Resource:** [resource_type]
**Compliance:** [framework]

**Fix Instructions:**
1. Open [file] at line [X]
2. [Specific fix for the technology]
3. Validate configuration

**Technology-Specific Actions:**
[If Terraform]:
- Run `terraform fmt`
- Run `terraform validate`
- Apply security best practice

[If Kubernetes]:
- Add security context
- Set resource limits
- Apply RBAC

[If Docker]:
- Use specific image tags
- Run as non-root
- Minimize layers

**Verification:**
- Re-scan with MCP tool
- Test in dev environment
- Check compliance status
```

### Step 6: Include Compliance Mapping
Add section for each framework:
- CIS Benchmark violations
- NIST controls gaps
- Security best practices

### Step 7: Create JSON Summary
Generate: `${SCAN_TIME}_iac_summary.json`

```json
{
  "scan_type": "iac",
  "files_by_type": {
    "terraform": N,
    "kubernetes": N,
    "docker": N
  },
  "compliance_status": {
    "CIS": "X violations",
    "NIST": "Y violations"
  },
  "severity_breakdown": {}
}
```

## Success Criteria
- All IaC files identified and scanned
- Technology-specific fixes provided
- Compliance mapping completed
- Report follows directory structure

## Error Handling
- If no IaC files: Report "No infrastructure code found"
- If MCP tool fails: Alert configuration issue
- Fallback to pattern-based checks if needed

## Priority Focus
1. Security groups with 0.0.0.0/0
2. Missing encryption settings
3. Overly permissive IAM roles
4. Public resource exposure
5. Missing security headers

## Output
Display: `IaC report generated with [N] findings requiring remediation`
