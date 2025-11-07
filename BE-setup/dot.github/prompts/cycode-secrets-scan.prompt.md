---
mode: 'agent'
description: 'Perform comprehensive secrets scan on changed files and generate remediation report'
---

# Cycode Secrets Scan with MCP Tools

Execute a security scan for exposed secrets using Cycode MCP tools.

## Context Required
- Current branch name
- Changed files in working tree
- Recent commit history

## Task: Scan for Exposed Secrets

### Step 1: Gather Uncommitted Files (Working Tree Only)
Collect ONLY files currently modified or newly added but not yet committed.
1. Branch name: `git rev-parse --abbrev-ref HEAD`
2. Uncommitted tracked changes: `git diff --name-only HEAD`
3. Untracked new files: `git ls-files --others --exclude-standard`
4. Combine + de-duplicate.
5. Exclude ONLY obvious large/binary or generated/vendor paths: images (`*.png,*.jpg`), other media (audio/video), and build/vendor artifacts (`dist/`, `build/`, `node_modules/`, `.venv/`, `__pycache__/`). DO NOT exclude Python settings or config files (e.g. `*settings*.py`, `local_*.py`, `*_config.py`) – these MUST be scanned because they often contain hard‑coded credentials. If a future exclusion pattern would omit such files, explicitly re-include them. (In short: exclude binaries & compiled/vendor junk; include all `.py`, `.env`, `.yml`, `.yaml`, `.json`, `.toml` unless binary.)
6. If env var `FULL_SCAN=1` provided (override) THEN include all repository files (excluding ignored patterns) and mark report with `(full scan override)`.
7. If resulting list empty → output: `No uncommitted changes detected. Secrets scan skipped.` and stop.

### Step 2: Execute MCP Secrets Scan (Working Tree Set)
Use the Cycode MCP tool `cycode_scan_secrets` with:
- `scan_type`: "secret"
- `paths`: [uncommitted + untracked file list]
- `include_history`: true (history limited to those files)
- `severity_threshold`: "ALL"

Notes:
- Do not include files from previous commits unless FULL_SCAN override is used.
- For >500 files, optionally batch ~250 and merge results.

### Step 3: Create Report Directory
```bash
SCAN_DATE=$(date +%Y-%m-%d)
SCAN_TIME=$(date +%H_%M)
BASE_DIR="ai-cycode-scans"
```
1. Create `ai-cycode-scans` directory if not exists
2. Find next run number for today
3. Create subdirectory: `${SCAN_DATE}.${RUN_NUMBER}`

### Step 4: Generate Markdown Report
Create file: `${SCAN_DATE}.${RUN_NUMBER}/${SCAN_TIME}_secrets.md`

Include these sections:
```markdown
# Cycode Secrets Scan Report
**Date:** [timestamp]
**Branch:** [branch name]
**Run:** [run number]

## Executive Summary
- Total files scanned: [count]
- Critical findings: [count]
- High findings: [count]

## Identified Issues
[For each finding, include:]
- Severity
- File and line number
- Secret type
- Detection rule

## Remediation Task List
```

### Step 5: Create Tasks for Each Finding
For each secret found, generate a task with:

```markdown
### Task [N]: Remove [secret_type] from [file]
**Priority:** [severity]
**Location:** [file:line]

**Actions:**
1. Remove secret from line [X]
2. Rotate the exposed credential
3. Store in secure vault (Vault/AWS Secrets Manager)
4. Update code to retrieve from vault

**Verification:**
- Re-scan file with MCP tool
- Confirm secret removed
- Test application functionality
```

### Step 6: Add JSON Summary
Create: `${SCAN_DATE}.${RUN_NUMBER}/${SCAN_TIME}_secrets_summary.json`

Structure:
```json
{
  "scan_type": "secrets",
  "date": "[ISO timestamp]",
  "branch": "[branch]",
  "total_findings": N,
  "severity_breakdown": {
    "critical": N,
    "high": N,
    "medium": N,
    "low": N
  }
}
```

## Success Criteria
- All changed files scanned
- Report generated in correct directory structure
- Each finding has actionable task
- JSON summary created
- Report location displayed to user

## Error Handling
- If MCP tool unavailable: Alert user to check configuration
- If no findings: Generate report stating "No secrets found"
- If directory creation fails: Use current directory

## Output Format
Display: `Report generated: ai-cycode-scans/[date].[run]/[time]_secrets.md`

## Notes
- NEVER display actual secret values
- Mask all sensitive information
- Prioritize CRITICAL and HIGH severity
- Include rotation instructions for each secret type
