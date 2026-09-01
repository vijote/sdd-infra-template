---
name: "speckit-checklist"
description: "Generate a custom technical checklist for infrastructure contracts, cloud security, and Kubernetes configurations."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/checklist.md"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before checklist generation)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_checklist` key
- Filter out hooks where `enabled` is explicitly `false`.
- For each executable hook, output the appropriate hook block based on its `optional` flag.

## Execution Steps

1. **Setup**: Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -Template checklist-template` from repo root and parse JSON for `FEATURE_DIR` and `TEMPLATE_CONTENT`.

2. **Load Directives & Context**:
   - Read `.specify/memory/constitution.md`
   - Read `spec.md`, `plan.md`, `tasks.md` from `FEATURE_DIR`

3. **Generate Technical Checklist**:
   - Create `FEATURE_DIR/checklists/` if needed.
   - Use filename based on technical focus: `terraform.md`, `cluster.md`, `security.md`, `database.md`, etc.
   - Format:
     ```markdown
     # [Category] Technical Checklist: [FEATURE NAME]

     ## 1. Contract & Schema Completeness
     - [ ] CHK001 Are all Terraform variables typed with default values and validation blocks?
     - [ ] CHK002 Are Kubernetes resources configured with explicit CPU/Memory limits and readiness probes?

     ## 2. Infrastructure & Security Boundaries
     - [ ] CHK003 Are IAM roles restricted with least-privilege policies?
     - [ ] CHK004 Are Security Group rules limited to required CIDRs and ports?
     - [ ] CHK005 Are cert-manager ClusterIssuers and ACME/TLS secrets configured securely?
     - [ ] CHK006 Are MySQL database credentials mounted from Kubernetes Secrets rather than plain-text configs?

     ## 3. Machine-Verifiable Acceptance Gates
     - [ ] CHK007 Can every acceptance scenario be validated using automated CLI commands?
     ```

4. **Report**: Output path to generated checklist, item count, and review summary.
