# SDD Infrastructure Template - Agent Quick Guide

## 📍 Key Locations

### Configuration
- `.specify/memory/constitution.md` - Project rules & tech stack
- `.specify/templates/` - All template files (spec, plan, tasks, checklist)
- `.specify/workflows/` - Workflow definitions

### Generated Files
- `specs/[###]/spec.md` - Feature specifications
- `plan.md` - Architecture delta & file impact matrix
- `tasks.md` - Task execution graph (DAG)
- `src/` - Generated infrastructure code

## 🚀 Agent Workflow

1. **speckit-specify** → Creates `specs/[###]/spec.md`
2. **speckit-plan** → Creates `plan.md` (reads spec.md)
3. **speckit-tasks** → Creates `tasks.md` (reads plan.md)
4. **speckit-implement** → Executes tasks (reads tasks.md)

## 🏗️ Technology Stack

- **AWS** + **Terraform** (no CloudFormation)
- **Kubeadm** + **Flannel CNI**
- **NGINX Ingress** + **cert-manager**
- **MySQL 8.0** + **EBS CSI**
- **GitHub OIDC** auth

## ⚡ Key Constraints

- **No tests** - Direct AWS validation only
- **Spec < 200 lines** - Keep concise
- **Tasks < 31 items** - Micro-DAG only
- **No Secrets Manager** - Use Kubernetes Secrets
- **No network tests** - Skip nslookup checks

## 📋 Template Files Reference

| Template | Purpose | Key Sections |
|----------|---------|--------------|
| `spec-template.md` | Feature specs | Infrastructure contracts, Kubernetes manifests, Data contracts |
| `plan-template.md` | Architecture delta | File impact matrix, Architectural boundaries, Verification gates |
| `tasks-template.md` | Execution graph | 6 stages: Terraform → Bootstrap → CNI → Ingress → Secrets → Workloads |
| `checklist-template.md` | Quality gates | Contract completeness, Security hygiene, Machine-verifiable criteria |

## 🔧 Common Validation Commands

```bash
# Terraform
terraform validate && terraform plan

# Kubernetes
kubectl get nodes -o wide
kubectl rollout status deployment/<name>

# Helm
helm lint <chart-path>
helm template <release> <chart-path>
```

## 🎯 Implementation Notes

- All AWS resources via Terraform modules
- Flannel uses VXLAN backend (default)
- NGINX Ingress with TLS termination
- MySQL with EBS gp3 storage
- GitHub OIDC for IAM roles
- Direct file operations only (no test files)