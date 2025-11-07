---
mode: agent
description: 'Execute scoped SCA remediation: upgrade vulnerable dependencies, handle license issues, update tracking'
---

# Cycode SCA Fix Prompt

Runs AFTER SCA remediation plan (`cycode-sca-plan.prompt.md`). Automates dependency upgrades & license mitigations.

## Scope Selection
- `critical-all`
- `critical-top:<K>`
- `high-top:<K>`
- `license-high-all`
- `license-mix:<high>:<med>`
- `id-list:SCA-CRIT-001,SCA-HIGH-004,LICENSE-HIGH-001`
- `auto` (Up to 10 vulnerabilities prioritizing Critical direct deps, then High direct, then Critical transitive)

## Inputs
- Run directory path
- `remediation-sca/dependencies.json`
- `remediation-sca/licenses.json`
- Individual `SCA-*` / `LICENSE-*` markdown files
- Package manager manifests (package.json, requirements.txt, etc.)

## Upgrade Strategy
1. For each selected vulnerability:
   - Determine ecosystem
   - Choose highest recommended patch version (prefer non-major if safe)
   - Modify manifest file
   - Update lock file (conceptual steps)
2. For license issues:
   - Propose alternative package name(s)
   - If alternative chosen update manifest & remove old dependency

## Ecosystem Actions (Conceptual)
| Ecosystem | Upgrade Action |
|-----------|----------------|
| npm/yarn | Modify package.json, run install to update lock |
| pip | Update requirements.txt pinned version |
| maven | Change `<version>` in pom.xml |
| go | `go get module@version` then tidy |
| ruby | Update Gemfile entry |

## Workflow
1. Parse dependency & license JSON indexes.
2. Resolve selected IDs.
3. For each dependency:
   - Locate manifest file.
   - Replace version string.
   - Annotate change (comment with SCA ID if format allows).
4. For license tasks:
   - Replace dependency OR tag for removal.
5. Update markdown file: status to in_progress then completed after simulated verification.
6. Update JSON indexes: status, history entry.
7. Append summary to `index.md` under progress section (optional block).

## Verification (Conceptual)
- Run tests (placeholder)
- Re-run SCA scan (targeted) -> vulnerability no longer appears
- If fix introduces new major break risk: mark as deferred instead

## Output Example
```
✅ SCA remediation batch
Scope: critical-top:5
Upgraded: 5 | License handled: 1 | Deferred: 0 | Errors: 0
```

## Error Handling
- Manifest not found: mark item deferred with reason
- Recommended version missing: mark deferred
- Duplicate version already patched: mark completed (idempotent)

## Safety
- Do not remove unrelated dependencies
- Maintain formatting of manifest
- Avoid broad upgrade (only targeted packages)

## Success Criteria
- Selected dependency versions bumped
- License tasks annotated or replaced
- Tracking artifacts updated

## Post-Run
- Commit: `sec: upgrade vulnerable dependencies batch <N>`
- Re-run `secure check sca`

