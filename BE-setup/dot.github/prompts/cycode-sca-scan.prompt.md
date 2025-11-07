---
mode: 'agent'
description: 'Analyze dependencies for vulnerabilities and generate Software Bill of Materials with remediation tasks'
---

# Cycode SCA Dependency Scan with MCP Tools

Execute Software Composition Analysis for vulnerability and license compliance using Cycode MCP tools.

## Context Required
- Package manifest files
- Dependency lock files
- Current dependency versions

## Task: Analyze Dependencies and Generate SBOM

### Step 1: Gather Uncommitted Dependency Manifests (Working Tree Only)
Limit scan strictly to manifests & lock files currently modified or newly added.

1. Uncommitted tracked changes: `git diff --name-only HEAD`
2. Untracked new files: `git ls-files --others --exclude-standard`
3. Combine + de-duplicate.
4. Filter to manifest/lock patterns:
  - Node.js: `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
  - Python: `requirements.txt`, `requirements/*.txt`, `Pipfile`, `pyproject.toml`, `poetry.lock`
  - Java: `pom.xml`, `build.gradle`, `gradle.lockfile`
  - Go: `go.mod`, `go.sum`
  - .NET: `*.csproj`, `packages.config`, `Directory.Packages.props`
  - Ruby: `Gemfile`, `Gemfile.lock`
5. Exclude vendor / generated directories: `node_modules/`, `.venv/`, `env/`, `site-packages/`, `target/`, `build/`.
6. If set empty → output: `No uncommitted dependency manifests detected; SCA scan skipped.` and stop unless `FULL_SCAN=1`.
7. If `FULL_SCAN=1` set: include all manifests across repo (excluding ignored dirs) and annotate report `(full scan override)`.

### Step 2: Execute MCP SCA Scan (Working Tree Set)
Use `cycode_scan_sca` with:
- `scan_type`: "sca"
- `scan_mode`: ["package-vulnerabilities", "license-compliance"]
- `paths`: [changed manifest & lock files]
- `generate_sbom`: true

Notes:
- Only scan manifests currently uncommitted unless FULL_SCAN override.
- If FULL_SCAN active: include all manifests.
- If a lock file changed but manifest did not (or vice versa), include its counterpart for accurate resolution.

### Step 3: Execute MCP SBOM Generation
Use `cycode_generate_sbom` tool:
- `format`: "json"
- `include_transitive`: true

### Step 4: Setup Report Structure
```bash
SCAN_DATE=$(date +%Y-%m-%d)
SCAN_TIME=$(date +%H_%M)
```
Create in: `ai-cycode-scans/${SCAN_DATE}.${RUN_NUMBER}/`

### Step 5: Build Vulnerability Report
Create: `${SCAN_TIME}_sca.md`

```markdown
# Cycode SCA Report
**Date:** [timestamp]
**Dependencies Scanned:** [count]
**Vulnerable Packages:** [count]

## Critical Vulnerabilities
[For each CVE with CVSS > 9.0]
- Package: [name@version]
- CVE: [CVE-ID]
- CVSS Score: [score]
- Fix Version: [version]

## Dependency Update Tasks
```

### Step 6: Generate Update Tasks
For each vulnerable package:

```markdown
### Task [N]: Update [package] - [CVE]
**Priority:** [Based on CVSS score]
**Current:** [version]
**Target:** [fix_version]
**Ecosystem:** [npm/pip/maven]

**Update Commands:**
[npm]: npm install [package]@[version]
[pip]: pip install [package]==[version]
[maven]: Update pom.xml
[go]: go get [package]@v[version]

**Implementation:**
1. Create branch: fix/[CVE]-[package]
2. Update package to [fix_version]
3. Update lock file
4. Run tests
5. Check for breaking changes

**Testing Checklist:**
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] No deprecation warnings
- [ ] Application starts correctly

**Rollback Plan:**
- Current version: [version]
- Backup lock files before update
- Revert command if needed
```

### Step 7: License Compliance Section
Add tasks for license issues:

```markdown
## License Compliance Issues
### Task [N]: Review [package] License
**Package:** [name]
**License:** [type]
**Action Required:** [Review/Replace/Exception]

Options:
1. Find alternative with compatible license
2. Request legal exception
3. Remove dependency
```

### Step 8: Generate SBOM Summary
Save SBOM to: `sbom_report.json`

Include:
- All components and versions
- License information
- Dependency tree
- Vulnerability status

### Step 9: Create Metrics JSON
Generate: `${SCAN_TIME}_sca_summary.json`

```json
{
  "scan_type": "sca",
  "total_dependencies": N,
  "direct_dependencies": N,
  "vulnerable_packages": N,
  "critical_cves": N,
  "license_issues": N,
  "ecosystems": {
    "npm": N,
    "python": N
  }
}
```

## Update Priority Matrix
Classify by urgency:
1. **Immediate (0-24h):** Known exploits, CVSS > 9.0
2. **High (48h):** CVSS 7.0-8.9
3. **Medium (1 week):** CVSS 4.0-6.9
4. **Low (next sprint):** CVSS < 4.0

## Success Criteria
- All dependencies scanned
- SBOM generated
- Update commands provided for each ecosystem
- Rollback plans included
- License issues identified

## Error Handling
- Missing package manager: Skip ecosystem
- No vulnerabilities: Report clean status
- SBOM generation fails: Create manual list

## Key Outputs
1. Vulnerability report with CVE details
2. SBOM in JSON format
3. Actionable update tasks
4. License compliance status

## Display
Show: `SCA complete: [N] vulnerabilities found, [M] require immediate action`
