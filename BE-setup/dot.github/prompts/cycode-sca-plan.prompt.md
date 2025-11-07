---
mode: agent
description: 'Process SCA dependency scan results, build remediation + upgrade tracking workspace'
---

# Cycode SCA Remediation Plan Generator

This prompt executes AFTER an SCA (Software Composition Analysis) scan (see `cycode-sca-scan.prompt.md`). It consumes scan output (vulnerable packages, CVEs, license issues, SBOM) and creates a standardized remediation & tracking workspace.

## Prerequisites
- SCA scan completed with outputs in: `ai-cycode-scans/<DATE>.<RUN>/`
- Markdown report: `<TIME>_sca.md`
- Summary JSON: `<TIME>_sca_summary.json`
- (Optional) SBOM: `sbom_report.json`

## Objectives
1. Normalize and persist dependency vulnerability dataset
2. Prioritize by severity + exploitability + dependency depth (direct vs transitive)
3. Track top impactful vulnerabilities per severity (up to 20) for focused remediation
4. Provide deterministic upgrade playbooks with rollback guidance
5. Distinguish vulnerability vs license compliance tasks
6. Offer machine-readable index for CI gates & automation
7. Enable auditable status transitions and historical progress metrics

## Step 1: Locate Latest SCA Scan
1. Identify newest `YYYY-MM-DD.N` scan directory in `ai-cycode-scans/`
2. Load `<TIME>_sca_summary.json`
3. Load `sbom_report.json` if present (for license & transitive mapping)
4. Load `<TIME>_sca.md` for contextual enhancement (if needed)

## Step 2: Parse and Enrich Findings
For each vulnerable component:
- Capture: package name, ecosystem, current version, affected version ranges, fixed version(s)
- Extract CVE list with CVSS (base + temporal if available)
- Determine dependency depth (direct / transitive)
- Derive exploitability flag (YES if known exploit reference or EPSS > threshold if provided)
- License (from SBOM) and license risk classification
- Compute risk score = (Severity Weight * 3) + (Exploitability ? 5 : 0) + (Direct? 3 : 0) + (Age Factor 0-2) + (Critical App Path? 3 : 0)
  - Severity Weight: Critical=5, High=3, Medium=2, Low=1
  - Age Factor: Older than 180d unresolved = 2, >90d = 1 else 0

## Step 3: Directory Structure
Create:
```
ai-cycode-scans/<DATE>.<RUN>/
├── <TIME>_sca.md
├── <TIME>_sca_summary.json
├── sbom_report.json (optional)
└── remediation-sca/
    ├── index.md                      # Master tracking dashboard
    ├── dependencies.json             # Machine index for vulnerabilities
    ├── licenses.json                 # Machine index for license issues
    ├── critical/
    │   ├── README.md
    │   ├── SCA-CRIT-001.md
    │   └── ...
    ├── high/
    │   ├── README.md
    │   └── SCA-HIGH-001.md
    ├── medium/
    │   ├── README.md
    │   └── SCA-MED-001.md
    ├── low/
    │   ├── README.md
    │   └── SCA-LOW-001.md
    ├── licenses/
    │   ├── README.md                 # Overview of license compliance tasks
    │   ├── LICENSE-HIGH-001.md
    │   └── ...
    └── reports/
        ├── TOP_OUTDATED.md           # Aggregated outdated libs (non-vuln)
        └── RISK_MATRIX.md            # Matrix summary
```

## Step 4: Selection Logic (Tracked Top)
For each severity bucket (C/H/M/L):
1. Filter out entries with no available fix (mark as `deferred` candidate)
2. Prioritize direct dependencies over transitive
3. Sort by composite risk score (Step 2)
4. Select top 20 (or fewer if fewer exist)
5. Remaining findings still stored in `dependencies.json` but not given individual remediation markdown initially

## Step 5: Master Index (`index.md`)
Template:
```markdown
# SCA Dependency Remediation Index
**Scan Date:** [date time]  
**Run:** <DATE>.<RUN>  
**Directory:** ./remediation-sca/

## Summary
| Severity | Total Found | Tracked (Top) | Completed | In Progress | Pending | Deferred | False Positives |
|----------|-------------|---------------|-----------|-------------|---------|----------|-----------------|
| Critical | [C_total]   | [C_tracked]   | 0         | 0           | [C_tracked] | 0        | 0               |
| High     | [H_total]   | [H_tracked]   | 0         | 0           | [H_tracked] | 0        | 0               |
| Medium   | [M_total]   | [M_tracked]   | 0         | 0           | [M_tracked] | 0        | 0               |
| Low      | [L_total]   | [L_tracked]   | 0         | 0           | [L_tracked] | 0        | 0               |

**Overall Progress:** 0 / ([C_tracked]+[H_tracked]+[M_tracked]+[L_tracked]) = 0%

## License Issues Summary
| Risk | Total | Tracked | Completed | Pending | Deferred |
|------|-------|---------|-----------|---------|----------|
| High | [LicHigh] | [LicHighTracked] | 0 | [LicHighTracked] | 0 |
| Medium | [LicMed] | [LicMedTracked] | 0 | [LicMedTracked] | 0 |
| Low | [LicLow] | [LicLowTracked] | 0 | [LicLowTracked] | 0 |

## Navigation
- [Critical](./critical/README.md)
- [High](./high/README.md)
- [Medium](./medium/README.md)
- [Low](./low/README.md)
- [License Issues](./licenses/README.md)
- [Risk Matrix](./reports/RISK_MATRIX.md)

## Critical Dependencies (Tracked)
- [ ] SCA-CRIT-001: [package@version] - [CVE]
- [ ] SCA-CRIT-002: [package@version] - [CVE]

## SLA Guidance
- Critical: upgrade < 48h
- High: < 7 days
- Medium: < 30 days
- Low: next sprint / opportunistic

## Status Legend
Pending | In Progress | Completed | Deferred | False Positive

## Last Updated
[timestamp]
```

## Step 6: Machine Index (`dependencies.json`)
Schema snippet:
```json
{
  "scan_metadata": {"date": "[ISO]", "run": "<DATE>.<RUN>", "source_report": "<TIME>_sca.md"},
  "statistics": {"critical": C_total, "high": H_total, "medium": M_total, "low": L_total},
  "tracked": {"critical": C_tracked, "high": H_tracked, "medium": M_tracked, "low": L_tracked},
  "dependencies": [
    {
      "id": "SCA-CRIT-001",
      "package": "lodash",
      "ecosystem": "npm",
      "current_version": "4.17.15",
      "fixed_versions": ["4.17.21"],
      "cves": ["CVE-XXXX-YYYY"],
      "cvss": 9.8,
      "severity": "CRITICAL",
      "direct": true,
      "status": "pending",
      "license": "MIT",
      "license_risk": null,
      "exploit_available": true,
      "risk_score": 23,
      "recommended_version": "4.17.21",
      "upgrade_blockers": [],
      "history": [ {"date": "[ISO]", "status": "pending"} ]
    }
  ]
}
```

## Step 7: License Issues (`licenses.json`)
Schema snippet:
```json
{
  "scan_metadata": {"date": "[ISO]", "run": "<DATE>.<RUN>"},
  "issues": [
    {
      "id": "LICENSE-HIGH-001",
      "package": "some-lib",
      "ecosystem": "pip",
      "current_version": "1.2.3",
      "license": "AGPL-3.0",
      "risk": "HIGH",
      "status": "pending",
      "recommended_action": "Replace with permissive alternative",
      "alternatives": ["other-lib"],
      "history": [ {"date": "[ISO]", "status": "pending"} ]
    }
  ]
}
```

## Step 8: Severity README Template
Example (`critical/README.md`):
```markdown
# Critical Dependency Vulnerabilities (Top 20)
| ID | Package | Current | Fix | CVEs | Status | Direct | Effort |
|----|---------|---------|-----|------|--------|--------|--------|
| [SCA-CRIT-001](./SCA-CRIT-001.md) | lodash | 4.17.15 | 4.17.21 | CVE-XXXX-YYYY | Pending | Yes | 15m |
```

## Step 9: Individual Package Remediation File
Filename: `SCA-<SEVERITY>-NNN.md`
```markdown
# SCA-<SEVERITY>-NNN: [package] Upgrade

## Status
**Current:** Pending  
**Severity:** <SEVERITY>  
**Direct Dependency:** Yes/No  
**Exploit Available:** Yes/No  
**Effort Estimate:** [e.g., 1h]

## Package Details
- **Ecosystem:** npm / pip / maven / go / rubygems
- **Current Version:** x.y.z
- **Fixed Versions:** v1, v2
- **Recommended Target:** vX.Y.Z
- **CVEs:** CVE-XXXX-YYYY (CVSS 9.8)
- **Transitive Path (if transitive):** root -> A -> B -> [package]

## Risk Assessment
- **Impact:** [RCE / Data leak / DoS]
- **Likelihood:** [High/Med/Low]
- **Overall Risk:** [Derived]
- **Exploit References:** [links]

## Upgrade Plan
1. Create branch: `dep/upgrade/[package]-[version]`
2. Apply upgrade command (examples below)
3. Run test suite
4. Validate build & lint
5. Smoke test critical flows
6. Update changelog / security notes

## Commands (Examples)
```bash
# npm
yarn add [package]@[version]
# pip
pip install [package]==[version]
# maven (pom.xml)
# Update <version> tag then:
mvn -q -DskipTests dependency:tree
```

## Breaking Change Review
- [ ] Check upstream release notes
- [ ] Review deprecations
- [ ] Run type checks / static analysis

## Rollback Plan
- Revert commit OR reinstall previous version
- Restore lock file from pre-upgrade snapshot

## Verification Checklist
- [ ] CVE no longer reported by SCA scan
- [ ] Tests passing
- [ ] No new high severity introduced
- [ ] Performance unaffected
- [ ] Documentation updated

## Status History
| Date | Status | By | Notes |
|------|--------|----|-------|
| [date] | Pending | System | Imported from scan |

## Notes
[Additional context]
```

## Step 10: License Issue File Template
Filename: `LICENSE-<RISK>-NNN.md`
```markdown
# LICENSE-<RISK>-NNN: [package] License Review

## Status
**Current:** Pending  
**Risk:** HIGH/MEDIUM/LOW  
**License:** [License Type]

## Issue Summary
Package uses [license] which is incompatible with policy due to [reason].

## Recommended Actions
- Option 1: Replace with [alternative]
- Option 2: Seek legal exception (ticket: LEGAL-XXX)
- Option 3: Remove usage (if low value)

## Decision Log
| Date | Decision | By | Notes |
|------|----------|----|-------|
| [date] | Pending | System | Imported from scan |
```

## Step 11: Reports
`reports/RISK_MATRIX.md` summarizing distribution (direct vs transitive, exploitability). Example snippet:
```markdown
| Severity | Direct | Transitive | Exploit Known | No Fix Available |
|----------|--------|-----------|---------------|------------------|
| Critical | 3      | 5         | 2             | 1                |
```

## Step 12: Append Note to Original Scan Report
Append to `<TIME>_sca.md`:
```markdown
---
## Remediation Plan Generated (SCA)
Artifacts in: ./remediation-sca/
Start with Critical upgrades: [link](./remediation-sca/critical/README.md)
```

## Step 13: Console Output
```
✅ SCA remediation plan created
• Vulnerable packages: [N] (Critical: C | High: H | Medium: M | Low: L)
• Tracked: [T]
• License issues tracked: [LicTracked]
Location: ai-cycode-scans/<DATE>.<RUN>/remediation-sca/
Next: Open remediation-sca/index.md and begin with SCA-CRIT-001
```

## Status Workflow Rules
Same as other plans: pending → in_progress → completed | deferred | false_positive
- `deferred`: justify (e.g., no fix, low exploitability) + re-review date
- `false_positive`: requires evidence (scanner misclassification / unreachable code)

## Automation Hooks
- CI fails if new Critical remains `pending` after grace window
- Scheduled job recomputes closure velocity
- Commit message referencing ID auto-updates status after merge

## Error Handling
- Missing summary: abort with instruction to run scan
- No vulnerabilities: create minimal index stating "No vulnerable packages detected"
- SBOM absent: license section still generated using partial data

## Success Criteria
- Directories created
- Index + JSON indices valid
- Individual remediation files for tracked vulnerabilities
- License issues separated
- Original scan report appended

## Maintenance Guidelines
1. Update individual `SCA-*` or `LICENSE-*` file
2. Sync status in `index.md` & `dependencies.json` / `licenses.json`
3. Add history entry on change

## Alignment Table
| Dimension | SCA Plan | Secrets Plan | IaC Plan | SAST Plan (target) |
|-----------|----------|--------------|----------|--------------------|
| Directory | remediation-sca | remediation | remediation-iac | remediation-sast |
| Machine Index | dependencies.json | vulnerabilities.json | misconfigurations.json | findings.json |
| IDs | SCA-<SEV>-NNN | VULN-<SEV>-NNN | IAC-<SEV>-NNN | SAST-<SEV>-NNN |
| Extra Group | licenses/* | n/a | compliance/* | CWE/OWASP mapping |

## Notes
- Prefer smallest safe upgrade (avoid major unless required)
- Batch related minor bumps when low risk
- Monitor transitive risk after direct fixes
