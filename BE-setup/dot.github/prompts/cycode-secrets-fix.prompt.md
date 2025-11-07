---
mode: agent
description: 'Execute scoped secrets remediation: remove hardcoded secrets, rotate credentials, update plan tracking'
---

# Cycode Secrets Fix Prompt

Runs AFTER secrets plan generation (`cycode-secrets-plan.prompt.md`). Applies targeted fixes to pending secret findings and updates tracking artifacts.

## Scope Selection
Accept one of:
- `critical-all` (all pending Critical)
- `critical-top:<K>` (first K Critical by order in index.md)
- `high-top:<K>`
- `severity-mix:<crit>:<high>:<med>` (e.g., 3:5:2)
- `id-list:VULN-CRIT-001,VULN-HIGH-004,...`
- `auto` (prioritize Critical then High until daily budget reached — default budget 8 items)

If none provided → `auto`.

## Inputs Required
- Path to scan run directory (e.g., `ai-cycode-scans/2025-11-04.3/`)
- `remediation/` subfolder must exist with:
  - `index.md`
  - `vulnerabilities.json`
  - Severity folders containing `VULN-*` files

## Workflow
1. Parse `vulnerabilities.json` → build in-memory list of pending items.
2. Apply scope filter.
3. For each selected item:
   - Identify file & line.
   - Load source file content.
   - Mask secret pattern (never log full value).
   - Perform replacement strategy:
     - Remove literal.
     - Add environment variable reference or secret manager retrieval.
   - If language is Python:
     - Replace `API_KEY = "hardcoded"` with `API_KEY = os.getenv("API_KEY")` (ensure `import os`).
   - If JavaScript/TypeScript:
     - Replace `const token = "hardcoded"` with `const token = process.env.TOKEN;`.
   - If Dockerfile:
     - Remove `ENV SECRET=hardcoded` lines; require runtime injection.
   - If config (YAML/JSON):
     - Replace value with `${ENV_VAR}` placeholder.
4. Write file changes.
5. Update corresponding `VULN-*.md`:
   - Status → `in_progress` then `completed` after verification.
   - Append history row.
6. Update `index.md` checkboxes (switch `[ ]` → `[x]`).
7. Update `vulnerabilities.json`:
   - Set `status` to `completed`.
   - Add history entry with timestamp + action summary.
8. Generate verification mini-scan (optional) for modified files only; if clean → finalize, else revert status to `in_progress` with note.

## Rotation & Credential Notes
- For each secret type: append note to vulnerability file listing rotation ticket (placeholder). If rotation cannot be automated, mark `notes` entry.

## Error Handling
- File missing: mark item `deferred` with note.
- Line not found: attempt fuzzy search; if fail, mark `false_positive` candidate for manual review.
- Permission error: abort and report.

## Logging (Internal)
- Output summary: processed count, completed, skipped, deferred.

## Output Summary Format
```
✅ Secrets remediation run complete
Scope: critical-top:5
Processed: 5 | Completed: 5 | Deferred: 0 | Errors: 0
Updated artifacts: index.md, vulnerabilities.json, VULN-* files
```

## Success Criteria
- All selected secret literals removed.
- Environment-based retrieval present.
- Tracking artifacts updated consistently.
- No secret value echoed in logs.

## Safety Guards
- Never print original secret.
- Keep only last 4 masked chars for identification (e.g., `****ABCD`).
- Do not overwrite unrelated lines.

## Post-Run Recommendations
- Commit with message: `sec: remove hardcoded secrets (batch X)`.
- Run `secure check secrets` to confirm clean state.

