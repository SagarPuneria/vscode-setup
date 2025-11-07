---
mode: agent
description: 'Execute scoped IaC remediation: apply config hardening, update plan tracking'
---

# Cycode IaC Fix Prompt

Runs AFTER IaC remediation plan (`cycode-iac-plan.prompt.md`). Applies fixes to misconfigurations and updates tracking.

## Scope Selection
- `critical-all`
- `critical-top:<K>`
- `high-top:<K>`
- `mix:<crit>:<high>:<med>`
- `id-list:IAC-CRIT-001,IAC-HIGH-002,...`
- `auto` (Critical then High until 10 items)

## Inputs
- Run directory: `ai-cycode-scans/<DATE>.<RUN>/`
- `remediation-iac/misconfigurations.json`
- Individual `IAC-*` markdown files

## Fix Strategy by Resource Type
| Resource | Common Issue | Fix Pattern |
|----------|--------------|-------------|
| Terraform aws_security_group | 0.0.0.0/0 ingress | Restrict CIDR / remove rule |
| Terraform aws_s3_bucket | Missing encryption | Add `server_side_encryption_configuration` |
| Kubernetes Deployment | Privileged container | Add `securityContext` least privilege |
| Kubernetes Pod | No resource limits | Add `resources.requests/limits` |
| Dockerfile | Running as root | Add non-root USER |
| Terraform aws_iam_policy | Wildcard actions | Scope down actions |

## Workflow
1. Parse machine index -> select pending items by severity.
2. For each item:
   - Read file & locate line region.
   - Apply transformation template.
   - Insert missing blocks (e.g., add encryption stanza).
   - Format (terraform fmt / ensure YAML indentation).
3. Update `IAC-*` file status: in_progress → completed.
4. Update checklist in `index.md`.
5. Update `misconfigurations.json` status + history.
6. Optionally run targeted validation:
   - Terraform: `terraform validate` (conceptual; not executed here).
   - Kubernetes: `kubectl apply --dry-run=client` (conceptual).
7. If validation concept fails: revert status to in_progress and add note.

## Error Handling
- Line not found: fuzzy search by resource block start.
- File immutable / locked: mark `deferred`.
- Unsupported resource type: add note and skip.

## Output Example
```
✅ IaC remediation run
Scope: mix:3:4:2
Processed: 9 | Completed: 8 | Deferred: 1
Changes: terraform(5) kubernetes(3) docker(1)
```

## Safety
- Avoid destructive changes (no automatic deletion of resources).
- Restrict only obvious broad exposures (0.0.0.0/0, wildcard IAM).
- Do not add provider blocks.

## Success Criteria
- Selected misconfigurations adjusted.
- Tracking artifacts synchronized.
- No syntax errors introduced.

## Post-Run
- Commit: `sec: harden IaC batch <N>`
- Re-run `secure check iac`.

