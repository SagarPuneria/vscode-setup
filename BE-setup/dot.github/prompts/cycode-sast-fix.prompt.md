---
mode: agent
description: 'Execute scoped SAST remediation: apply secure code patches, update tracking and mapping artifacts'
---

# Cycode SAST Fix Prompt

Runs AFTER SAST remediation plan (`cycode-sast-plan.prompt.md`). Applies code fixes for vulnerable patterns and updates remediation tracking.

## Scope Selection
- `critical-all`
- `critical-top:<K>`
- `high-top:<K>`
- `mix:<crit>:<high>:<med>`
- `owasp:A03:<K>` (top K injection issues)
- `cwe:CWE-079:<K>`
- `id-list:SAST-CRIT-001,SAST-HIGH-004,...`
- `auto` (Prioritize Critical reachable injection findings then High XSS until 12 items)

## Inputs
- Run directory path
- `remediation-sast/findings.json`
- Severity README & `SAST-*` files
- Source code files referenced
- (Optional) SARIF for rule metadata

## Patch Strategy Patterns
| Vulnerability | Language | Before | After |
|---------------|----------|--------|-------|
| SQL Injection | python | f-string query | parameterized query |
| XSS | js | innerHTML assignment | textContent or sanitized HTML |
| Command Injection | java | exec with concatenation | ProcessBuilder args |
| Path Traversal | python | open(user_input) | validated & joined path |
| Hardcoded Crypto Key | any | key literal | secure retrieval (env/secret mgr) |

## Workflow
1. Parse machine index and filter scope.
2. For each finding:
   - Read file and lines (line_start-line_end).
   - Capture original snippet (store masked hash for audit).
   - Apply transformation per vulnerability type.
   - Insert required imports (e.g., `import os`).
   - Ensure idempotency (skip if already secure pattern).
3. Update individual `SAST-*` file: status → in_progress then completed after pseudo-verification.
4. Update `findings.json`: status + history entry.
5. Regenerate snippet hash.
6. Update `index.md` checklist.
7. Append summary to `OWASP_MATRIX.md` / `CWE_DISTRIBUTION.md` (optional delta section).

## Verification (Conceptual)
- Static re-scan of modified files
- Run unit tests targeting patched logic
- Negative test: attempted injection fails

## Output Example
```
✅ SAST remediation run
Scope: critical-top:5
Patched: 5 | Skipped (already secure): 0 | Deferred: 0
```

## Error Handling
- Line mismatch: search by approximate code pattern
- File missing: mark deferred
- Unsupported language: skip with note

## Safety
- Minimal patch: avoid refactors beyond vulnerability fix
- Preserve formatting and comments
- Do not alter logic unrelated to security change

## Success Criteria
- Selected vulnerable snippets replaced with secure patterns
- Tracking artifacts consistent
- No introduction of syntax errors

## Post-Run
- Commit: `sec: patch code vulnerabilities batch <N>`
- Re-run `secure check sast`

## Notes
- Future enhancement: integrate real SARIF diff & unit test harness
- Consider adding regression test generator per vulnerability class

