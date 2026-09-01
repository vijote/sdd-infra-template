---
name: "speckit-clarify"
description: "Identify underspecified technical areas in the current infrastructure spec by asking targeted clarification questions."
compatibility: "Requires spec-kit project structure with .specify/ directory"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/clarify.md"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Pre-Execution Checks

**Check for extension hooks (before clarification)**:
- Check if `.specify/extensions.yml` exists in the project root.
- If it exists, read it and look for entries under the `hooks.before_clarify` key
- Filter out hooks where `enabled` is explicitly `false`.
- For each executable hook, output the appropriate hook block based on its `optional` flag.

## Outline

Goal: Detect and eliminate technical ambiguities, missing cloud parameters, or undefined infrastructure contracts in `spec.md` before planning.

1. **Setup**: Run `.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly` from repo root and parse `FEATURE_SPEC`.

2. **Load Directives**: Load `.specify/memory/constitution.md`.

3. **Technical Ambiguity Scan**: Scan `spec.md` across technical dimensions:
   - **Network Topology & CIDRs**: VPC, subnet allocations, pod/service CIDRs
   - **Compute & Node Sizing**: EC2 instance types, ASG scaling limits, kubelet configurations
   - **Storage & CSI**: EBS volume types, IOPS, StorageClass retention & mount options
   - **Security & IAM**: Instance profile policies, security group port allowances, cert-manager ClusterIssuer challenge types
   - **Database & Stateful Services**: MySQL replica configuration, persistence volume sizing, root/app credentials handling
   - **Acceptance Verifiability**: Ensure all criteria map to concrete CLI test commands

4. **Sequential Questioning Loop**:
   - Ask up to 5 targeted technical multiple-choice or short-answer questions.
   - Present a recommended option with concise engineering rationale.
   - Example table:
     | Option | Technical Choice | Engineering Tradeoff / Impact |
     | :--- | :--- | :--- |
     | A | [e.g. AWS EBS gp3 StorageClass] | [Dynamic provisioning, custom IOPS/throughput] |
     | B | [e.g. HostPath / Local PV] | [Node-locked, non-portable for multi-node] |
   - Integrate approved answers directly into `spec.md` under relevant contract and acceptance criteria sections.
   - Save `spec.md` and re-validate `checklists/requirements.md`.

## Mandatory Post-Execution Hooks

Check if `.specify/extensions.yml` exists in the project root.
- If it exists, execute hooks under `hooks.after_clarify` as defined in the standard SpecKit protocol.

## Completion Report

Report number of questions clarified, sections updated in `spec.md`, and readiness for `/speckit-plan`.
