---
mode: agent
description: 'Process SAST scan results to generate code vulnerability remediation & tracking workspace'
---

# Cycode SAST Remediation Plan Generator

This prompt executes AFTER a SAST scan (`cycode-sast-scan.prompt.md`). It builds a structured remediation workspace incorporating CWE / OWASP mappings, code fix playbooks, and progress tracking.

## Prerequisites
- SAST scan completed with outputs in: `ai-cycode-scans/<DATE>.<RUN>/`
- Markdown report: `<TIME>_sast.md`
- Summary JSON: `<TIME>_sast_summary.json`
- (Optional) SARIF: `sast_sarif.json`

## Objectives
1. Normalize code vulnerability findings with CWE + OWASP metadata
2. Prioritize by severity + exploitability + data sensitivity + reachability (if available)
3. Track top 20 per severity (Critical / High / Medium / Low)
4. Provide language-specific secure code patterns & patch guidance
5. Maintain human dashboard + machine index for automation
6. Enable auditable status lifecycle & history
7. Append plan reference to original scan report

## Step 1: Locate Latest SAST Scan
1. Pick most recent `YYYY-MM-DD.N` directory under `ai-cycode-scans/`
2. Load `<TIME>_sast_summary.json`
3. Load `sast_sarif.json` (use for precise locations + rule ids if present)
4. Load `<TIME>_sast.md`

## Step 2: Parse & Enrich Findings
For each finding extract:
- ID (to generate)
- File path, line (start-end if range)
- Severity (Critical/High/Medium/Low)
- CWE ID(s)
- OWASP Category (Axx)
- Rule ID / Engine
- Snippet (sanitized)
- Suggested fix (from engine if any)
Enrich with:
- Language (python / javascript / etc.)
- Reachability flag (dynamic evidence / call graph) if data present
- Data sensitivity (contains PII? secret patterns? config?) heuristic
- Risk score = SeverityWeight + (Reachable?3:0) + (Sensitive?2:0) + (InjectionType?3:0) + (AuthContext?2:0)
  - SeverityWeight: Critical=8, High=5, Medium=3, Low=1

## Step 3: Directory Structure
```
ai-cycode-scans/<DATE>.<RUN>/
├── <TIME>_sast.md
├── <TIME>_sast_summary.json
├── sast_sarif.json (optional)
└── remediation-sast/
    ├── index.md
    ├── findings.json                 # Machine index (all)
    ├── cwe/                          # Aggregated by CWE
    │   ├── CWE-079.md
    │   └── ...
    ├── owasp/                        # Aggregated by OWASP category
    │   ├── A01.md
    │   └── ...
    ├── critical/
    │   ├── README.md
    │   ├── SAST-CRIT-001.md
    │   └── ...
    ├── high/
    │   ├── README.md
    │   └── SAST-HIGH-001.md
    ├── medium/
    │   ├── README.md
    │   └── SAST-MED-001.md
    ├── low/
    │   ├── README.md
    │   └── SAST-LOW-001.md
    └── reports/
        ├── OWASP_MATRIX.md
        ├── CWE_DISTRIBUTION.md
        └── RISK_MATRIX.md
```

## Step 4: Selection Logic (Top Tracking)
- Group by severity
- Sort each group by risk score descending
- Pick top 20 per severity (or fewer if not available)
- Remaining findings remain only in `findings.json`

## Step 5: Master Index (`index.md`)
```markdown
# SAST Code Vulnerability Remediation Index
**Scan Date:** [date time]  
**Run:** <DATE>.<RUN>  
**Directory:** ./remediation-sast/

## Summary
| Severity | Total Found | Tracked (Top) | Completed | In Progress | Pending | Deferred | False Positives |
|----------|-------------|---------------|-----------|-------------|---------|----------|-----------------|
| Critical | [C_total]   | [C_tracked]   | 0         | 0           | [C_tracked] | 0        | 0               |
| High     | [H_total]   | [H_tracked]   | 0         | 0           | [H_tracked] | 0        | 0               |
| Medium   | [M_total]   | [M_tracked]   | 0         | 0           | [M_tracked] | 0        | 0               |
| Low      | [L_total]   | [L_tracked]   | 0         | 0           | [L_tracked] | 0        | 0               |

**Overall Progress:** 0 / ([C_tracked]+[H_tracked]+[M_tracked]+[L_tracked]) = 0%

## OWASP Distribution
| OWASP Category | Count | Tracked |
|----------------|-------|---------|
| A01 | N | N |
| A03 | N | N |

## CWE Distribution (Top 5)
| CWE | Count | Example IDs |
|-----|-------|-------------|
| CWE-079 | N | SAST-HIGH-003 |

## Navigation
- [Critical](./critical/README.md)
- [High](./high/README.md)
- [Medium](./medium/README.md)
- [Low](./low/README.md)
- [CWE Mapping](./cwe/)
- [OWASP Mapping](./owasp/)
- [Risk Matrix](./reports/RISK_MATRIX.md)

## Critical Findings (Tracked)
- [ ] SAST-CRIT-001: [Summary] (file.py:123)
- [ ] SAST-CRIT-002: [Summary]

## SLA Guidance
- Critical: fix < 72h
- High: < 14 days
- Medium: < 30 days
- Low: Opportunistic / backlog grooming

## Status Legend
Pending | In Progress | Completed | Deferred | False Positive

## Last Updated
[timestamp]
```

## Step 6: Machine Index (`findings.json`)
Schema snippet:
```json
{
  "scan_metadata": {"date": "[ISO]", "run": "<DATE>.<RUN>", "source_report": "<TIME>_sast.md"},
  "statistics": {"critical": C_total, "high": H_total, "medium": M_total, "low": L_total},
  "tracked": {"critical": C_tracked, "high": H_tracked, "medium": M_tracked, "low": L_tracked},
  "findings": [
    {
      "id": "SAST-CRIT-001",
      "file": "src/app/service.py",
      "line_start": 120,
      "line_end": 122,
      "severity": "CRITICAL",
      "cwe": ["CWE-089"],
      "owasp": ["A03:Injection"],
      "rule_id": "python.sql.injection",
      "language": "python",
      "status": "pending",
      "snippet_hash": "abc123",
      "risk_score": 21,
      "reachable": true,
      "sensitive_data": true,
      "exploit_notes": "Unsafely concatenated SQL",
      "history": [{"date": "[ISO]", "status": "pending"}]
    }
  ]
}
```

## Step 7: Severity README Example
`critical/README.md`:
```markdown
# Critical Code Vulnerabilities (Top 20)
| ID | File | CWE | OWASP | Status | Risk | Effort |
|----|------|-----|-------|--------|------|--------|
| [SAST-CRIT-001](./SAST-CRIT-001.md) | service.py:120 | CWE-089 | A03 | Pending | 21 | 45m |
```

## Step 8: Individual Finding File
Filename: `SAST-<SEVERITY>-NNN.md`
```markdown
# SAST-<SEVERITY>-NNN: [Short Vulnerability Title]

## Status
**Current:** Pending  
**Severity:** <SEVERITY>  
**Reachable:** Yes/No  
**Sensitive Data:** Yes/No  
**Effort Estimate:** [e.g., 45m]

## Finding Details
- **File:** path/to/file.py
- **Lines:** 120-122
- **Language:** python
- **CWE:** CWE-089 (SQL Injection)
- **OWASP:** A03: Injection
- **Rule ID:** python.sql.injection
- **Description:** Raw user input concatenated into SQL query.

## Insecure Snippet
```python
query = f"SELECT * FROM users WHERE id = {user_input}"  # vulnerable
cursor.execute(query)
```

## Secure Pattern
```python
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_input,))
```

## Remediation Steps
1. Parameterize query
2. Add input validation (type & range)
3. Add regression test for injection attempt
4. Re-run SAST scanner
5. Mark status updated

## Verification Checklist
- [ ] SAST scan clean for this finding
- [ ] New test added & passes
- [ ] No performance regression
- [ ] Code review approved

## Risk Assessment
- **Impact:** Data exfiltration possible
- **Likelihood:** High
- **Overall Risk:** Critical

## Status History
| Date | Status | By | Notes |
|------|--------|----|-------|
| [date] | Pending | System | Imported from scan |

## Notes
[Additional context]
```

## Step 9: CWE / OWASP Aggregation Files
Generate one file per CWE & OWASP category listing associated finding IDs and short descriptions.

## Step 10: Reports
- `OWASP_MATRIX.md`: table counts per OWASP
- `CWE_DISTRIBUTION.md`: histogram style summary
- `RISK_MATRIX.md`: severity vs risk score quadrant counts

## Step 11: Append to Original Report
Append to `<TIME>_sast.md`:
```markdown
---
## Remediation Plan Generated (SAST)
Artifacts: ./remediation-sast/
Begin with high-risk Critical items: [link](./remediation-sast/critical/README.md)
```

## Step 12: Console Output
```
✅ SAST remediation plan created
• Total findings: [N] (Critical: C | High: H | Medium: M | Low: L)
• Tracked: [T]
Location: ai-cycode-scans/<DATE>.<RUN>/remediation-sast/
Next: Open remediation-sast/index.md and start with SAST-CRIT-001
```

## Status Workflow Rules
Same shared model (pending → in_progress → completed | deferred | false_positive)
- `deferred`: allowed for low exploitability or legacy code slated for deprecation
- `false_positive`: must capture reasoning & reviewer

## Automation Hooks
- SARIF ingestion → IDE annotations
- CI gate: block merge if new Critical introduced vs previous baseline
- Auto-close if diff removes vulnerable lines & scan re-run passes

## Error Handling
- Missing summary: instruct to run SAST scan first
- SARIF absent: proceed without rule metadata enrichment
- No findings: minimal index with clean status message

## Success Criteria
- Directory & files created
- Machine index with all findings
- Individual files for tracked top items
- CWE/OWASP mapping artifacts generated
- Original report appended

## Alignment Table
| Dimension | SAST Plan | SCA Plan | Secrets Plan | IaC Plan |
|-----------|-----------|----------|--------------|----------|
| Directory | remediation-sast | remediation-sca | remediation | remediation-iac |
| Machine Index | findings.json | dependencies.json | vulnerabilities.json | misconfigurations.json |
| IDs | SAST-<SEV>-NNN | SCA-<SEV>-NNN | VULN-<SEV>-NNN | IAC-<SEV>-NNN |
| Extra Grouping | CWE/OWASP | Licenses | n/a | Compliance |

## Maintenance Guidelines
1. Update individual `SAST-*` file & snippet
2. Sync status in `index.md` & `findings.json`
3. Append to history on each status transition
4. Periodic re-scan to validate completed issues remain resolved

## Notes
- Favor smallest safe fix; avoid over-refactors that inflate risk
- Add regression tests for each injection / auth / XSS class of bug
- Keep code examples sanitized (no secrets / sensitive tokens)
