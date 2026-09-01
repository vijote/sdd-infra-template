---
name: "speckit-analyze"
description: "Perform a non-destructive cross-artifact consistency and technical contract analysis across spec.md, plan.md, and tasks.md."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/analyze.md"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before analysis)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_analyze` key
- Filter out hooks where `enabled` is explicitly `false`.
- For each executable hook, output the appropriate hook block based on its `optional` flag.

## Goal

Identify inconsistencies, missing file mappings, DAG dependency breaks, and unmapped acceptance criteria across the three core artifacts (`spec.md`, `plan.md`, `tasks.md`) before implementation.

## Operating Constraints

**STRICTLY READ-ONLY**: Do **not** modify any files. Output a structured analysis report.
**Constitution Authority**: `.specify/memory/constitution.md` is non-negotiable. Violations are automatically CRITICAL.

## Execution Steps

1. **Initialize Analysis Context**:
   - Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks` from repo root.
   - Load `spec.md`, `plan.md`, `tasks.md`, and `.specify/memory/constitution.md`.

2. **Cross-Artifact Consistency & Contract Passes**:
   - **Pass A: File Impact & Task Coverage**: Verify every file in `plan.md`'s File Impact Matrix maps to at least one 1:1 task in `tasks.md`.
   - **Pass B: Acceptance Criteria Coverage**: Verify every Acceptance Criterion (`AC-###`) in `spec.md` has a mapped Stage 5 verification task in `tasks.md`.
   - **Pass C: DAG Integrity & Dependencies**: Check that task dependencies `(Depends on Txxx)` in `tasks.md` form a valid acyclic graph (no cycles, valid upstream task IDs, Stage prerequisite consistency).
   - **Pass D: Constitution Compliance**:
     - Check for narrative fluff / non-technical storytelling (must be zero).
     - Check artifact line counts (must be <200 lines).
     - Verify acceptance criteria are machine-verifiable CLI commands.

3. **Produce Analysis Report**:
   Output a compact table:
   ```markdown
   ## Technical Specification Analysis Report

   | ID | Category | Severity | Location | Summary | Recommendation |
   | :--- | :--- | :--- | :--- | :--- | :--- |
   | A1 | Unmapped Contract | HIGH | plan.md:L12 | `k8s/apps/mysql/pvc.yaml` missing task in tasks.md | Add Stage 4 task for PVC creation |

   **Coverage Metrics**:
   - Total Technical Contracts: N
   - Total Acceptance Criteria: N
   - Total Micro-DAG Tasks: N
   - Contract Task Coverage: 100%
   - Constitution Compliance: PASS/FAIL
   ```

4. **Suggest Next Actions**: Suggest running `/speckit-implement` if clean, or specific artifact adjustments if critical issues are detected.
