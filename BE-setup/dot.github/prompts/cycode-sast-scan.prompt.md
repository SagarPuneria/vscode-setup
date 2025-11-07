---
mode: 'agent'
description: 'Perform static code analysis for security vulnerabilities with language-specific remediation'
---

# Cycode SAST Code Analysis with MCP Tools

Execute Static Application Security Testing on source code using Cycode MCP tools.

## Context Required
- Source code files
- Programming languages used
- OWASP Top 10 mapping

## Task: Analyze Code for Security Vulnerabilities

### Step 1: Gather Uncommitted Source Files (Working Tree Only)
Collect only currently modified or newly added source files to perform a focused incremental SAST.

1. Uncommitted tracked changes: `git diff --name-only HEAD`
2. Untracked new files: `git ls-files --others --exclude-standard`
3. Combine + de-duplicate → candidate set.
4. Filter by language extensions:
  - JavaScript/TypeScript: `*.js`, `*.jsx`, `*.ts`, `*.tsx`
  - Python: `*.py`
  - Java/Kotlin: `*.java`, `*.kt`
  - C# / VB: `*.cs`, `*.vb`
  - Go: `*.go`
  - PHP: `*.php`
  - Ruby: `*.rb`
5. Exclude test/support patterns unless `INCLUDE_TESTS=1`: `**/tests/**`, `**/__tests__/**`, `**/spec/**`.
6. Exclude generated/build/vendor dirs: `dist/`, `build/`, `node_modules/`, `.venv/`, `__pycache__/`.
7. If list empty → output: `No uncommitted source changes detected. SAST scan skipped.` and stop unless `FULL_SCAN=1`.
8. If `FULL_SCAN=1` override set: include all source files (excluding vendor) and annotate report `(full scan override)`.

### Step 2: Execute MCP SAST Scan (Working Tree Set)
Use `cycode_scan_sast` with:
- `scan_type`: "sast"
- `paths`: [changed source files]
- `languages`: [auto-detected from extensions]
- `owasp_mapping`: true
- `cwe_classification`: true

Notes:
- Limit scan to uncommitted files unless FULL_SCAN override.
- If >800 files, batch (~300 each) and aggregate results.

### Step 3: Initialize Report Structure
```bash
SCAN_DATE=$(date +%Y-%m-%d)
SCAN_TIME=$(date +%H_%M)
```
Directory: `ai-cycode-scans/${SCAN_DATE}.${RUN_NUMBER}/`

### Step 4: Create Vulnerability Report
Generate: `${SCAN_TIME}_sast.md`

```markdown
# Cycode SAST Report
**Date:** [timestamp]
**Files Analyzed:** [count]
**Languages:** [list]

## Security Findings by Severity

### CRITICAL Issues
[Group by vulnerability type]

### HIGH Issues
[Group by CWE category]
```

### Step 5: Generate Fix Tasks with Code Examples
For each finding, create detailed task:

```markdown
### Task [N]: Fix [vulnerability] in [file]
**Severity:** [CRITICAL/HIGH/MEDIUM/LOW]
**Location:** [file:line]
**CWE:** [CWE-ID]
**OWASP:** [Category]

**Vulnerability:**
[Description of the security issue]

**Secure Code Pattern:**
```

Include language-specific fix:

**For JavaScript (XSS example):**
```javascript
// Vulnerable:
element.innerHTML = userInput;

// Secure:
element.textContent = userInput;
// Or use DOMPurify for HTML
element.innerHTML = DOMPurify.sanitize(userInput);
```

**For Python (SQL Injection example):**
```python
# Vulnerable:
query = f"SELECT * FROM users WHERE id = {user_id}"

# Secure:
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

**For Java (Command Injection example):**
```java
// Vulnerable:
Runtime.getRuntime().exec("ping " + userInput);

// Secure:
ProcessBuilder pb = new ProcessBuilder("ping", userInput);
pb.start();
```

### Step 6: Map to OWASP Top 10
Create table showing distribution:

```markdown
## OWASP Top 10 Coverage
| Category | Count | Priority |
|----------|-------|----------|
| A01: Broken Access Control | N | HIGH |
| A03: Injection | N | CRITICAL |
| A02: Cryptographic Failures | N | HIGH |
```

### Step 7: Add Remediation Steps
For each task, include:

```markdown
**Fix Steps:**
1. Locate vulnerable code at line [X]
2. Apply secure pattern shown above
3. Add input validation
4. Implement output encoding
5. Add security tests

**Validation:**
- Re-run SAST scan on file
- Test with malicious input
- Verify functionality preserved
- Add unit test for security

**References:**
- CWE: https://cwe.mitre.org/data/definitions/[ID].html
- OWASP: [relevant guide link]
```

### Step 8: Generate SARIF Output
Request MCP tool to generate SARIF format:
- Use for IDE integration
- Save as: `sast_sarif.json`

### Step 9: Create Summary JSON
Generate: `${SCAN_TIME}_sast_summary.json`

```json
{
  "scan_type": "sast",
  "languages_scanned": ["javascript", "python"],
  "total_findings": N,
  "severity_breakdown": {},
  "owasp_categories": {},
  "cwe_distribution": {},
  "files_affected": N
}
```

## Code Quality Checks
Also identify:
1. Hardcoded credentials
2. Weak cryptography
3. Path traversal
4. XXE vulnerabilities
5. Insecure deserialization

## Priority Matrix
Focus on exploitable vulnerabilities:
1. **Critical:** Remote code execution, SQL injection
2. **High:** Authentication bypass, XSS
3. **Medium:** Information disclosure
4. **Low:** Best practice violations

## Success Criteria
- All source files analyzed
- Language-specific fixes provided
- OWASP mapping completed
- SARIF report generated
- Code examples included

## Error Handling
- Unsupported language: Note in report
- MCP tool failure: Use fallback patterns
- No findings: Report clean status

## Output
Display: `SAST complete: [N] vulnerabilities found across [M] files. [X] require immediate fix.`
