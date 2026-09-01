---
name: "speckit-converge"
description: "Assess the current codebase against technical contracts and append any unmet infrastructure tasks to tasks.md."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/converge.md"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before convergence)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_converge` key
- Filter out hooks where `enabled` is explicitly `false`.
- For each executable hook, output the appropriate hook block based on its `optional` flag.

## Goal

Compare the existing Terraform code, Kubernetes manifests, Helm values, and scripts against `spec.md`, `plan.md`, and `tasks.md`. Identify missing files, incomplete configurations, unverified health gates, or unfulfilled acceptance criteria, and append them as new micro-DAG tasks at the bottom of `tasks.md`.

## Operating Constraints

**APPEND-ONLY TO TASKS.MD**:
- Do not modify `spec.md` or `plan.md`.
- Do not rewrite existing completed tasks in `tasks.md`.
- Append a new `## Stage N: Convergence` section to `tasks.md` with new sequential IDs `T{M+1:03d}`.
- If the codebase already fully converges, leave `tasks.md` untouched and report success.

## Execution Steps

1. **Load Context**: Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks`. Load `spec.md`, `plan.md`, `tasks.md`, and `.specify/memory/constitution.md`.

2. **Assess Technical Infrastructure & Artifacts**:
   - Inspect files defined in `plan.md`'s File Impact Matrix:
     - Check existence and completeness of Terraform files (`main.tf`, `variables.tf`, `outputs.tf`).
     - Check existence and validity of Kubernetes manifests, Helm values, and bootstrap scripts (`kubeadm`, `cert-manager`, `mysql`, `cni`, `ebs-csi`).
   - Classify findings:
     - `missing`: Required manifest/module is absent.
     - `partial`: Manifest/module exists but is missing required variables, health probes, or resources.
     - `unverified`: Acceptance criterion CLI verification has not been executed.

3. **Append Convergence Tasks** (if gaps detected):
   - Determine highest existing task ID `M` and next stage number `N`.
   - Append to `tasks.md`:
     ```markdown
     ## Stage N: Convergence
     - [ ] T026 [Stage N: Convergence] Create missing PVC in k8s/apps/mysql/pvc.yaml per spec.md §1.3 (missing) (Depends on T018)
     ```

4. **Report Outcome**:
   - Output summary of checked infrastructure files and findings.
   - If converged: Report "✅ Converged — all infrastructure contracts and acceptance gates are satisfied."
   - If tasks appended: Recommend running `/speckit-implement` to complete them.
