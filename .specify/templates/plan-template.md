# Architecture Delta: [FEATURE_NAME]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [specs/###-feature-name/spec.md]

## 1. Touch Points & File Impact Matrix

| File Path | Operation (Create/Modify/Delete) | Purpose / Exports |
| :--- | :--- | :--- |
| `terraform/modules/[module]/main.tf` | Create/Modify | Core AWS resource declarations |
| `terraform/modules/[module]/variables.tf` | Create/Modify | Input variables and validation rules |
| `terraform/modules/[module]/outputs.tf` | Create/Modify | Exported outputs for downstream modules |
| `terraform/environments/[env]/main.tf` | Modify | Root module instantiation & remote state |
| `k8s/bootstrap/[script].sh` | Create/Modify | Kubeadm initialization and join automation |
| `k8s/addons/[addon]/values.yaml` | Create/Modify | Helm values configuration (e.g. cert-manager) |
| `k8s/manifests/[workload]/` | Create/Modify | Kubernetes manifests (MySQL StatefulSet, PVC, Services) |

## 2. Architectural Boundaries & Dependency Flow

- **Infrastructure Layer (AWS & Terraform)**: [VPC, Subnets, IAM Instance Profiles, Security Groups, EC2 Compute]
- **Cluster Control Plane & Core Addons**: [kubeadm init/join, CNI Plugin, AWS EBS CSI Driver, StorageClass]
- **Platform Addons & Workloads**: [cert-manager ClusterIssuer, Ingress Controller, MySQL StatefulSet/Helm release]
- **Shared Dependencies**: [AWS provider version constraints, Kubernetes minor version, Helm repository versions]

## 3. Provisioning & Rollout Stages

1. **Stage 1 - Terraform IaC**: Apply network, IAM, compute, and security group infrastructure.
2. **Stage 2 - Kubeadm Bootstrap**: Execute control-plane initialization, token generation, and worker node joining.
3. **Stage 3 - Core Addons**: Deploy CNI (Calico/Cilium) and AWS EBS CSI storage driver.
4. **Stage 4 - Platform Services**: Deploy cert-manager with Let's Encrypt / Self-Signed ClusterIssuer.
5. **Stage 5 - State & Workloads**: Provision MySQL StatefulSet/Helm with persistent volume claims and credentials.

## 4. Verification Gates

- **IaC Verification**: `terraform validate && terraform plan`
- **Manifest Linting**: `helm lint [chart-path]` and `kubectl apply --dry-run=client -k [path]`
- **Cluster Health Gate**: `kubectl get nodes -o wide` (All nodes Ready)
- **Service Verification**: `kubectl rollout status` and health check commands
