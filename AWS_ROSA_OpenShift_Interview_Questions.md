# AWS ROSA (Red Hat OpenShift Service on AWS) – Complete Interview Question Bank
**Target Audience:** 3–4+ years experience DevOps / Cloud / SRE / Platform / OpenShift engineers  
**Scope:** All ROSA modules + OpenShift concepts + Real-time + Scenario + Troubleshooting  
**Source mix:** MNC interview patterns (TCS, Infosys, Wipro, Cognizant, Accenture, Deloitte, Capgemini, IBM, HCL, LTIM, Red Hat partners, Mphasis) + LinkedIn shared questions + real production ROSA incidents + Red Hat certification style.  
**Note:** No duplication with EC2 / VPC / IAM / S3 / EKS / Lambda / Route 53 / CloudWatch / CloudTrail banks. This bank focuses on **ROSA-specific and OpenShift-specific concepts** — upstream Kubernetes topics already covered in EKS bank.

---

## TABLE OF CONTENTS
1. ROSA Fundamentals
2. ROSA vs EKS vs Self-managed OpenShift
3. ROSA Classic vs ROSA with HCP (Hosted Control Planes)
4. Red Hat & AWS Joint Management Model
5. ROSA Subscription & Billing
6. Prerequisites & Account Setup
7. STS (Security Token Service) Mode
8. IAM Roles & OIDC Provider for ROSA
9. ROSA CLI (`rosa`)
10. OpenShift CLI (`oc`) vs kubectl
11. Cluster Creation (rosa CLI / OCM / Terraform)
12. Networking Architecture (Public vs PrivateLink)
13. VPC & Subnet Requirements
14. Cluster Admin vs Dedicated-Admin
15. Identity Providers (IDPs)
16. OpenShift Projects vs K8s Namespaces
17. Security Context Constraints (SCCs)
18. OpenShift Routes vs Kubernetes Ingress
19. DeploymentConfigs vs Deployments
20. BuildConfigs & ImageStreams
21. OpenShift Operators & OperatorHub
22. Cluster Operators
23. MachineSets & Machine API
24. MachineConfig & MachineConfigPools
25. Cluster Autoscaling (MachineAutoscaler)
26. Storage (EBS / EFS CSI on ROSA)
27. Monitoring Stack (Prometheus / Thanos)
28. Logging Stack (Loki / Vector / Fluentd)
29. OpenShift GitOps (ArgoCD based)
30. OpenShift Pipelines (Tekton)
31. OpenShift Service Mesh (Istio based)
32. OpenShift Serverless (Knative)
33. Upgrades & Lifecycle
34. Backup & Disaster Recovery (OADP)
35. Compliance & Security Operators
36. Cost Optimization on ROSA
37. Scenario-Based Questions
38. Troubleshooting Questions
39. Real-time Production Issues
40. DevOps / IaC Focused ROSA Questions

---

## 1. ROSA FUNDAMENTALS

**Q1. What is ROSA?**  
**Red Hat OpenShift Service on AWS** — fully managed turnkey OpenShift Container Platform offered jointly by Red Hat and AWS. Native AWS service (billed on AWS bill). Eliminates need to self-install OpenShift.

**Q2. What does "jointly managed" mean?**  
Both Red Hat SRE and AWS support teams manage the service. Single support ticket (via Red Hat or AWS) routes to appropriate team. 24×7 SRE monitoring + incident response — customer not responsible for control plane health.

**Q3. Why choose ROSA over self-managed OpenShift?**  
- No hardware/VMs to manage.  
- AWS-native billing (no separate Red Hat invoice typically).  
- Integrated AWS services (ALB, EBS, EFS, IAM).  
- SRE-managed control plane.  
- 99.95% SLA (Classic) / 99.95% (HCP).  
- Faster time-to-production (install in ~45 min vs days for IPI/UPI).

**Q4. What's the OCP version cadence on ROSA?**  
ROSA typically supports latest 3 OCP minor versions. New minor version released every ~4 months. Red Hat manages upgrades; you opt-in to schedule.

**Q5. ROSA regions:**  
Available in most commercial AWS regions. Check Red Hat's region availability matrix — not all regions/AZs support ROSA (especially HCP has narrower region coverage).

**Q6. Is ROSA FedRAMP / compliance certified?**  
ROSA is HIPAA eligible, SOC 2, ISO 27001 compliant. FedRAMP Moderate available in GovCloud (via ROSA on GovCloud). Enables regulated workloads.

**Q7. What Red Hat subscription do you need?**  
ROSA includes Red Hat entitlements in the hourly cost — you **don't** bring separate OpenShift subscriptions. For Red Hat add-on products (OpenShift AI, Advanced Cluster Manager), separate entitlements may apply.

**Q8. Which OS runs on ROSA nodes?**  
**Red Hat CoreOS (RHCOS)** — immutable, container-optimized OS managed by Machine Config Operator. Atomic updates via ostree. No traditional package manager on nodes.

---

## 2. ROSA vs EKS vs SELF-MANAGED OPENSHIFT

**Q9. ROSA vs EKS — detailed comparison:**  

| Dimension | ROSA | EKS |
|---|---|---|
| Kubernetes distribution | OpenShift (opinionated, secured) | Upstream Kubernetes |
| Control plane managed by | Red Hat + AWS SRE | AWS |
| Node OS | RHCOS (immutable) | AL2, AL2023, Bottlerocket, Ubuntu |
| Built-in CI/CD | OpenShift Pipelines (Tekton), BuildConfigs | Bring your own |
| Built-in GitOps | OpenShift GitOps (ArgoCD) | Install ArgoCD yourself |
| Built-in Service Mesh | OpenShift Service Mesh (Istio) | Install Istio/App Mesh |
| Built-in Registry | Internal Image Registry | Bring your own / ECR |
| Web console | Rich OpenShift Console | Basic AWS Console |
| Ingress | Routes (+ Ingress) | AWS LB Controller |
| Security | SCCs (stricter than PSA) | Pod Security Admission |
| Multi-tenancy | Projects (namespace + metadata) | Namespaces |
| Developer experience | Opinionated, dev-friendly | DIY |
| Pricing | Hourly + node cost | Control plane $0.10/hr + nodes |
| Lock-in | Tighter to OpenShift ecosystem | Upstream K8s portable |

**Q10. When choose ROSA over EKS?**  
- Team already knows OpenShift.  
- Need built-in CI/CD + GitOps + Service Mesh + Registry.  
- Strict security requirements (SCCs, FIPS, compliance operators).  
- Red Hat vendor relationship (enterprise support, indemnification).  
- Polyglot dev teams wanting self-service platform.

**Q11. When EKS is better than ROSA?**  
- Pure upstream Kubernetes preference.  
- Lower cost baseline for minimal workload.  
- No need for opinionated platform features.  
- Team prefers CNCF-native tooling.  
- Serverless use cases (Fargate fit).

**Q12. ROSA vs OpenShift on-prem (OCP) vs OpenShift Dedicated (OSD):**  
- **OCP on-prem** — you install on bare metal / vSphere / own cloud; fully self-managed.  
- **OSD** — Red Hat-managed on AWS/GCP (Red Hat's AWS account, you pay Red Hat).  
- **ROSA** — Red Hat-managed on your AWS account (you pay AWS).  
ROSA vs OSD: ROSA = native AWS integration + your AWS bill + BYOVPC; OSD = simpler if no AWS relationship.

**Q13. ROSA vs ARO (Azure Red Hat OpenShift):**  
Both jointly managed. ARO on Azure (jointly by Microsoft + Red Hat). Architecture + concepts very similar. Cloud-specific integrations differ (IAM vs Entra ID, EBS vs Azure Disk).

---

## 3. ROSA CLASSIC vs ROSA with HCP (HOSTED CONTROL PLANES)

**Q14. What are the two ROSA architectures?**  
- **ROSA Classic** — full control plane (3 master nodes + 2-3 infra nodes) runs inside your AWS account.  
- **ROSA with HCP (Hosted Control Planes)** — control plane runs in a Red Hat-managed AWS account; only worker nodes in yours. Launched GA 2024.

**Q15. Classic vs HCP — comparison:**

| | ROSA Classic | ROSA with HCP |
|---|---|---|
| Control plane location | Your AWS account (3 masters + infra nodes) | Red Hat-managed account |
| Minimum node count | 3 masters + 2 infra + 2 workers = 7 EC2 | 2 workers |
| Cluster creation time | ~45 min | ~10-15 min |
| Cost baseline | Higher (you pay for masters/infra) | Lower (no master cost) |
| Control plane EC2 visible | Yes, in your bill | No |
| Multi-AZ master | Required (3 AZs) | Managed by Red Hat |
| Upgrade speed | Longer | Faster |
| Available regions | Most | Fewer (growing) |
| SRE access | Same | Same |

**Q16. Why HCP is game-changing?**  
- **Cost savings** — no master/infra node cost (significant for many small clusters).  
- **Faster provisioning** — control plane pre-warmed pool.  
- **Denser fleet economics** — feasible to run many small clusters (dev/QA/prod/per-team).  
- **Faster upgrades** — control plane upgrade decoupled from data plane.

**Q17. When to still use Classic?**  
- Region not yet HCP-supported.  
- Compliance requires control plane in customer's account.  
- Specific networking constraints.  
- Existing investment in Classic.

**Q18. Upgrade path between Classic and HCP?**  
**No in-place migration.** Need blue-green: create new HCP cluster, migrate workloads (Velero/OADP + manifests), decommission Classic.

---

## 4. RED HAT & AWS JOINT MANAGEMENT MODEL

**Q19. Responsibility split — who manages what?**  
- **Red Hat SRE** — control plane, add-on operators, upgrade orchestration.  
- **AWS** — underlying EC2, EBS, VPC, AWS services.  
- **Customer** — workloads, application data, node group sizing, IAM in their account, network config.

**Q20. How does SRE access your cluster?**  
Via `sre` role and OpenShift Managed Openshift SRE bot. Break-glass access logged in CloudTrail. Customers can request access revocation or auditing.

**Q21. Can you have cluster-admin on ROSA?**  
Yes — grant `cluster-admin` role to specific users. However, some operators/CRDs managed by SRE are protected (`managed.openshift.io` prefix typically).

**Q22. What can't you do on ROSA that you could on self-managed OCP?**  
- Modify platform operators directly (SRE-managed).  
- Change master node types (managed).  
- Disable metrics/logging for platform.  
- Bypass SCCs for privileged workloads (without SRE approval).  
- Install MachineConfigs that break core functionality.

**Q23. SLA for ROSA:**  
99.95% monthly uptime for API + integrated console. Credits if breached. Different SLA tiers per support level.

**Q24. Support plan options:**  
- **Standard** — business hours.  
- **Premium** — 24×7 with faster response.  
- Premium required for production workloads typically.

---

## 5. ROSA SUBSCRIPTION & BILLING

**Q25. ROSA pricing model:**  
- **ROSA service fee** — per vCPU-hour of worker nodes.  
- **AWS infrastructure** — EC2 + EBS + LB + data transfer at standard AWS rates.  
- Billed through AWS invoice (single pane).

**Q26. Classic ROSA cost components (example small cluster):**  
- 3 master nodes (m5.2xlarge): ~$0.40/hr × 3.  
- 2-3 infra nodes (r5.xlarge): ~$0.25/hr × 3.  
- Worker nodes: per workload.  
- ROSA service fee: ~$0.17/vCPU-hour (check current pricing).  
- EBS + LB + NAT data.  
Baseline ~$1,500-2,000/month before workloads.

**Q27. HCP cost savings scenario:**  
Classic baseline: ~$1,500/month (masters + infra). HCP baseline: ~$250/month (HCP service fee + 2 worker nodes). For dev clusters, 80%+ savings.

**Q28. Reserved Instances / Savings Plans on ROSA:**  
Yes — ROSA nodes are regular EC2 → covered by Compute Savings Plans and RIs. Big savings for production baseline.

**Q29. Marketplace subscription:**  
ROSA purchased via AWS Marketplace; consolidated on AWS bill. No separate Red Hat invoice typically. Private Offers available for enterprise.

---

## 6. PREREQUISITES & ACCOUNT SETUP

**Q30. Prerequisites for installing ROSA:**  
1. AWS account with appropriate limits (EC2 vCPU, Elastic IPs, VPC).  
2. ROSA service enabled (`rosa verify quota`).  
3. Red Hat account linked (free; needed for OCM access).  
4. `rosa` CLI installed.  
5. IAM user/role with required permissions (AdministratorAccess or scoped policy).  
6. Service-Linked roles for ELB, AutoScaling, etc.

**Q31. Service quotas to pre-check:**  
- Running On-Demand Standard instances (vCPU).  
- Elastic Load Balancers.  
- VPC per region.  
- Route Tables, Elastic IPs.  
- Internet Gateways.  
`rosa verify quota --region <r>` automates check.

**Q32. Enable ROSA in account — steps:**  
1. AWS Console → ROSA page → click "Enable ROSA."  
2. Accept Red Hat terms.  
3. Link Red Hat account.  
4. Run `rosa init` (legacy mode — pre-STS).  
5. Verify with `rosa verify permissions`.

---

## 7. STS (SECURITY TOKEN SERVICE) MODE

**Q33. What is STS mode in ROSA?**  
**Security Token Service** deployment model (default since 2022). Uses short-lived credentials via IAM roles + OIDC instead of IAM user access keys. Modern, more secure, recommended.

**Q34. STS mode vs non-STS (legacy "IAM user") mode:**  
- **Legacy (non-STS)** — installer creates IAM user with access keys. Keys long-lived. Deprecated.  
- **STS** — installer creates IAM roles + OIDC provider. Credentials assumed via `AssumeRoleWithWebIdentity`. Auto-rotated. Least privilege.

**Q35. STS components:**  
- **Account roles** — per-AWS-account (create once, reuse across clusters): `Installer`, `ControlPlane`, `Worker`, `Support`.  
- **Operator roles** — per-cluster (for cluster operators to interact with AWS).  
- **OIDC Provider** — per cluster.

**Q36. Creating account roles:**  
```bash
rosa create account-roles --mode auto --yes
# or manual to review CloudFormation
rosa create account-roles --mode manual
```

**Q37. Creating operator roles:**  
```bash
rosa create operator-roles --cluster my-cluster --mode auto --yes
rosa create oidc-provider --cluster my-cluster --mode auto --yes
```
Required after cluster creation for operators to function.

**Q38. Why OIDC provider per cluster?**  
Each cluster has unique OIDC issuer URL → service accounts assume operator roles via `AssumeRoleWithWebIdentity`. Similar to IRSA in EKS but for OpenShift operators (ingress, storage, image registry, etc.).

**Q39. IAM permissions needed to install ROSA:**  
Red Hat publishes JSON policy (scoped, not AdministratorAccess). Covers EC2, VPC, IAM, Route 53, ELB, S3 (internal registry), KMS.

**Q40. Rotating OIDC keys:**  
OIDC keys auto-managed. Manual rotation via `rosa rotate`:  
```bash
rosa rotate keys --cluster my-cluster
```

---

## 8. IAM ROLES & OIDC PROVIDER DEEP DIVE

**Q41. Four account-level STS roles — what do they do?**  
- **Installer** — assumed by installer to provision AWS resources.  
- **Support** — assumed by Red Hat SRE for support tasks (break-glass).  
- **ControlPlane** — used by master nodes for AWS API calls.  
- **Worker** — used by worker nodes.

**Q42. Operator roles (examples):**  
- `cluster-image-registry-operator` — access S3 for registry storage.  
- `cluster-ingress-operator` — create LBs / Route 53 records.  
- `cluster-storage-operator` (EBS/EFS CSI) — create volumes.  
- `cluster-network-operator` — manage VPC resources.  
- `machine-api` — create/manage EC2 via MachineSets.

**Q43. How to audit STS role usage:**  
CloudTrail events for `sts:AssumeRoleWithWebIdentity` + `sts:AssumeRole`. Includes session name (pod identity).

**Q44. Cross-account setup for ROSA?**  
Not directly supported — ROSA cluster IAM must be in same account. Workloads can AssumeRole cross-account (your apps decide).

---

## 9. ROSA CLI (`rosa`)

**Q45. What does `rosa` CLI do?**  
Red Hat-provided CLI (Go binary) for ROSA lifecycle. Wraps OCM API. Install via Red Hat docs.

**Q46. Common rosa commands:**  
```bash
rosa login                    # OCM login
rosa list clusters
rosa create cluster --sts ...
rosa describe cluster <name>
rosa delete cluster <name> --yes
rosa create machinepool ...
rosa list idps --cluster <>
rosa create idp --cluster <> --type htpasswd ...
rosa logs install -c <cluster> --watch
rosa upgrade cluster -c <cluster> --version 4.16.x
rosa grant user cluster-admin --user <> --cluster <>
```

**Q47. `rosa verify ...` subcommands:**  
- `rosa verify quota` — AWS service quotas.  
- `rosa verify permissions` — IAM permissions on STS roles.  
- `rosa verify network` — subnet/VPC reachability pre-flight.

**Q48. rosa describe cluster — important fields:**  
- `API URL` — Kubernetes API endpoint.  
- `Console URL` — OpenShift web console.  
- `Nodes` — infra/worker breakdown.  
- `State` — ready, installing, error.  
- `Channel Group` — stable, candidate, fast.

---

## 10. OPENSHIFT CLI (`oc`) vs KUBECTL

**Q49. What is `oc`?**  
OpenShift CLI — superset of `kubectl`. All `kubectl` commands work + OpenShift-specific extensions (`oc login`, `oc new-project`, `oc rsh`, `oc new-app`, `oc expose`, `oc rollout`, `oc adm`).

**Q50. `oc login`:**  
Authenticates against cluster API using IDP credentials. Stores token in kubeconfig. `oc login --web` opens browser SSO.

**Q51. `oc new-project`:**  
Creates Project (+ namespace + metadata). Automatically sets you as admin, creates default SCC bindings.

**Q52. Handy `oc` commands interviewers quiz:**  
```bash
oc projects                      # list accessible projects
oc project <name>                # switch context
oc status                        # project summary
oc get all                       # all resources (Deployments, Services, Routes, Builds...)
oc rsh <pod>                     # remote shell into pod
oc logs <pod> -f                 # tail logs
oc describe sa default           # default service account
oc expose svc <name>             # create Route
oc rollout restart deploy/<name>
oc adm top nodes                 # like kubectl top
oc adm must-gather               # collect diagnostics bundle
oc debug node/<node>             # debug pod on node
```

**Q53. `oc adm must-gather` — when asked:**  
Collects comprehensive cluster diagnostics (events, configs, logs). Required artifact for Red Hat support cases. Output to local directory; upload to case.

**Q54. `oc debug node/<name>`:**  
Launches privileged debug pod on specified node (like `chroot /host`). Useful for CoreOS inspection without SSH.

---

## 11. CLUSTER CREATION

**Q55. Create cluster via rosa CLI (HCP, STS):**  
```bash
rosa create cluster \
  --cluster-name my-hcp \
  --sts --mode auto \
  --hosted-cp \
  --region us-east-1 \
  --subnet-ids subnet-pub,subnet-priv \
  --replicas 2 \
  --compute-machine-type m5.xlarge
```

**Q56. Create Classic cluster:**  
```bash
rosa create cluster \
  --cluster-name my-classic \
  --sts --mode auto \
  --multi-az \
  --region us-east-1 \
  --version 4.16.10 \
  --compute-nodes 3
```

**Q57. BYOVPC (Bring Your Own VPC) vs Red Hat-created VPC:**  
- **Red Hat-created** — rosa creates VPC, subnets, NAT, IGW.  
- **BYOVPC** — use existing VPC (enterprise norm). Subnets must satisfy CIDR, tagging, NAT/IGW requirements. Pass `--subnet-ids`.

**Q58. BYOVPC subnet requirements:**  
- Public subnets for LBs (if public cluster).  
- Private subnets for nodes.  
- Multi-AZ (at least 2 AZ for HCP, 3 AZ recommended for Classic multi-AZ).  
- NAT Gateway for egress from private.  
- IGW attached to VPC.  
- Tags (`kubernetes.io/role/elb`, `kubernetes.io/role/internal-elb`).  
- `enableDnsSupport` + `enableDnsHostnames`.  
- DHCP options with AmazonProvidedDNS.

**Q59. Red Hat OCM (OpenShift Cluster Manager):**  
Web UI at `console.redhat.com/openshift` — create/view clusters, add IDPs, monitor upgrades. Alternative to rosa CLI (wraps same API).

**Q60. Terraform for ROSA:**  
- `rhcs` Terraform provider (HashiCorp) for cluster resources.  
- Combine with `aws` provider for VPC/STS roles.  
- Community examples by Red Hat on GitHub.

**Q61. Cluster creation time typical:**  
- Classic: 30-45 min.  
- HCP: 10-15 min.  
Progress visible via `rosa logs install -c <cluster> --watch`.

**Q62. Cluster creation failures — common causes:**  
- Service quota exceeded.  
- Subnet missing tags.  
- IAM roles missing perms.  
- VPC DNS not enabled.  
- Insufficient AZ coverage.  
- Permission boundary blocking.  
- SCP denying required actions.

---

## 12. NETWORKING ARCHITECTURE

**Q63. Public vs Private cluster:**  
- **Public** — API + Ingress exposed via public LBs.  
- **Private** — API + Ingress only via private subnets (VPC-internal).  
- **PrivateLink** — API via PrivateLink endpoint; even more locked down.

**Q64. PrivateLink ROSA:**  
API server accessible only via VPC endpoint (`com.amazonaws.<region>.openshift`). No internet exposure. Enterprise standard for regulated environments.

**Q65. Pod/Service CIDRs:**  
Default:  
- Pod CIDR: `10.128.0.0/14` (OVN-Kubernetes default).  
- Service CIDR: `172.30.0.0/16`.  
- Machine CIDR: from your VPC subnets.  
Don't overlap with VPC CIDR or peered VPCs.

**Q66. OVN-Kubernetes vs OpenShift SDN:**  
- **OVN-Kubernetes** — default since OCP 4.12+ (modern, eBPF-adjacent, IPv6, L7 network policies).  
- **OpenShift SDN** — legacy, being removed.  
ROSA clusters use OVN by default now.

**Q67. NetworkPolicy vs AdminNetworkPolicy:**  
- **NetworkPolicy** — namespace-scoped (standard Kubernetes).  
- **AdminNetworkPolicy** (ANP, OCP 4.14+) — cluster-scoped, higher-priority, can't be overridden by namespace policies. Platform team enforces baselines.

**Q68. EgressIP and EgressFirewall:**  
- **EgressIP** — pods in namespace egress from specific static IP (useful for 3rd-party allowlists).  
- **EgressFirewall** — restrict outbound destinations per namespace (CIDR/DNS allowlist).

**Q69. Hybrid networking:**  
Classic ROSA supports cluster-wide VPN/Direct Connect via VPC networking. No direct ROSA-specific hybrid config — standard AWS networking applies.

---

## 13. VPC & SUBNET REQUIREMENTS

**Q70. Minimum subnet count:**  
- Classic multi-AZ: 6 subnets (3 public + 3 private, across 3 AZs).  
- Classic single-AZ: 2 subnets (1 pub + 1 priv).  
- HCP: 2 subnets (pub + priv), multi-AZ recommended.  
Private cluster: public subnets not required (but usually have 1 NAT public).

**Q71. Subnet CIDR sizing:**  
Each subnet should be large enough for all pods + nodes. `/24` = 251 usable IPs. For dense clusters or with IRSA-style features, go `/23` or `/22`.

**Q72. Can I add more subnets post-install?**  
For MachineSets in new AZ — yes, create MachineSet pointing to new subnet. Cluster network CIDRs fixed at install.

**Q73. CIDR conflicts — common mistake:**  
Machine CIDR (VPC) accidentally overlaps with pod CIDR. Pods can't reach some VPC resources. Plan carefully; document CIDR allocation.

---

## 14. CLUSTER ADMIN vs DEDICATED-ADMIN

**Q74. `cluster-admin` role:**  
Full admin privileges like self-managed OCP — can modify any resource. Granted selectively (SRE oversight for ROSA-managed resources still applies).

**Q75. `dedicated-admin` role:**  
ROSA-specific elevated role — admin over workload namespaces but **not** ROSA platform namespaces (`openshift-*`, `kube-*`). Safer default for customer admins.

**Q76. When to use which?**  
- `dedicated-admin` — daily operations, workload mgmt.  
- `cluster-admin` — infra-level changes (CRDs, cluster operators, RBAC).

**Q77. Granting cluster-admin:**  
```bash
rosa grant user cluster-admin --user <username> --cluster <cluster>
# behind scenes: creates ClusterRoleBinding
```

**Q78. `system:admin`:**  
Automatic admin via certificate (internal service). You generally don't use this directly on ROSA.

---

## 15. IDENTITY PROVIDERS (IDPs)

**Q79. Supported IDP types on ROSA:**  
- **htpasswd** — file-based, dev/test only.  
- **GitHub** — OAuth with GitHub org/team.  
- **GitLab** — OAuth.  
- **Google** — Google Workspace.  
- **LDAP** — corporate AD/LDAP.  
- **OpenID Connect** — Okta, Auth0, Keycloak, Azure AD/Entra.

**Q80. Add GitHub IDP example:**  
```bash
rosa create idp --cluster my-cluster --type github \
  --client-id <> --client-secret <> \
  --organizations <gh-org-name>
```

**Q81. Multiple IDPs:**  
Supported — users choose IDP at login. Common: OIDC (SSO) for employees + htpasswd break-glass for emergencies.

**Q82. kubeadmin user:**  
Temporary super-admin created at install time. Strongly recommended to delete after setting up real IDPs:  
```bash
oc delete secret kubeadmin -n kube-system
```

**Q83. RBAC model on ROSA:**  
Same as OCP — `ClusterRole`, `ClusterRoleBinding`, `Role`, `RoleBinding`. OpenShift adds convenience roles (`view`, `edit`, `admin`, `cluster-admin`, `self-provisioner`).

**Q84. Groups integration:**  
OIDC IDP can map groups claim → OpenShift groups. Then `RoleBinding` to groups instead of individual users. Essential for large teams.

---

## 16. OPENSHIFT PROJECTS vs NAMESPACES

**Q85. Project vs Namespace:**  
Project = Kubernetes Namespace + OpenShift metadata (display name, description, annotations). `oc new-project` creates both; `kubectl create namespace` creates only namespace (missing some OpenShift integration).

**Q86. Why Projects are useful:**  
- Self-service creation (with `self-provisioner` role).  
- Project-level quotas / limits.  
- Pre-populated default ServiceAccounts, RoleBindings, SCC assignments.  
- Visible in web console with friendly names.

**Q87. Project creation:**  
```bash
oc new-project my-app --description "My app" --display-name "My Application"
```

**Q88. Default project quotas (platform-defined):**  
ROSA defines default `ClusterResourceQuota` / `LimitRange` for projects. Customer admins can customize per project.

**Q89. Project lifecycle:**  
- Create → pod scheduled → deletes cleanup labels.  
- `oc delete project` triggers finalizers.  
- Stuck-in-Terminating projects: investigate finalizers, delete resources first.

---

## 17. SECURITY CONTEXT CONSTRAINTS (SCCs)

**Q90. What is an SCC?**  
**Security Context Constraints** — OpenShift's stricter-than-PSA pod admission control. Defines what Security Context a pod can request (runAsUser, privileged, capabilities, volume types, SELinux, etc.).

**Q91. Default SCCs in OpenShift:**  
- `restricted-v2` (default) — most locked down, assigns random UID.  
- `restricted` — legacy, stricter.  
- `anyuid` — lets pod run as any UID.  
- `hostaccess` — allows host namespaces.  
- `hostmount-anyuid` — host mounts + any UID.  
- `hostnetwork` — host network.  
- `node-exporter` — prometheus exporter.  
- `nonroot` — any non-root UID.  
- `privileged` — most permissive (root, all caps, host access).

**Q92. SCC assignment:**  
Pod's ServiceAccount binds to SCC via `RoleBinding`/`ClusterRoleBinding` with `use` verb on SCC. OpenShift automatically picks highest-priority allowed SCC.

**Q93. Why `default` SCC fails for some workloads?**  
Containers running as root (e.g., nginx default) fail under `restricted-v2` (random UID). Fix: image runs as non-root OR grant `anyuid` SCC to service account.

**Q94. Granting SCC to a service account:**  
```bash
oc adm policy add-scc-to-user anyuid -z <sa-name> -n <namespace>
```

**Q95. Custom SCC:**  
```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: my-scc
runAsUser:
  type: MustRunAsRange
  uidRangeMin: 1000
  uidRangeMax: 2000
fsGroup:
  type: MustRunAs
# ... many other fields
```
Used for tight/tailored security policies.

**Q96. SCC priority & selection:**  
Multiple SCCs available → OpenShift picks by priority (integer). Higher priority wins. If no SCC allows, pod rejected.

**Q97. Migrating from Kubernetes to OpenShift — SCC gotcha:**  
Helm charts designed for PSA often fail on OCP. Fix: either update chart to be rootless OR grant appropriate SCC (not cluster-wide `anyuid`). Prefer fixing image.

**Q98. PodSecurityAdmission vs SCC:**  
Both enforced simultaneously in OCP 4.11+. PSA labels at namespace level; SCC at pod spec level. Must satisfy both. Some redundancy, but SCCs still primary.

---

## 18. OPENSHIFT ROUTES vs KUBERNETES INGRESS

**Q99. What is a Route?**  
OpenShift's L7 traffic ingress abstraction. Predates Ingress. Includes TLS termination, path/host routing, weighted routing.

**Q100. Route vs Ingress — comparison:**  

| Feature | Route | Ingress |
|---|---|---|
| Native to | OpenShift | Kubernetes |
| TLS termination | Edge, Passthrough, Re-encrypt | Usually edge (controller-specific) |
| Weighted traffic (canary) | Built-in | Via annotations/controller |
| Custom domains | Yes (cert in Route) | Yes |
| Default hostname | `<name>-<ns>.apps.<cluster>.<domain>` | Controller-dependent |
| Load Balancer | Managed by Ingress operator (HAProxy) | Depends on controller |

**Q101. Route TLS modes:**  
- **Edge** — TLS terminated at router (HAProxy), backend HTTP.  
- **Passthrough** — TLS passed to backend; router doesn't decrypt.  
- **Re-encrypt** — router decrypts, re-encrypts with backend's cert.

**Q102. Route example:**  
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-app
spec:
  host: app.apps.example.com
  to: { kind: Service, name: my-app, weight: 100 }
  port: { targetPort: 8080 }
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

**Q103. OpenShift Ingress Operator:**  
Manages `IngressController` CRD → deploys HAProxy router pods → creates AWS ELB for them. Routes / Ingress objects both served by same HAProxy.

**Q104. Multiple IngressControllers:**  
For separating internal vs external traffic. Create additional `IngressController` with different domain + endpointPublishingStrategy.

**Q105. Default Apps domain:**  
ROSA assigns `<cluster>.<uuid>.<region>.openshiftapps.com`. Custom domain: configure DNS CNAME + IngressController.

**Q106. Using AWS ALB Controller on ROSA:**  
Possible — install aws-load-balancer-controller operator. For most ROSA users, default HAProxy Routes suffice. ALB used when ALB features (WAF, OIDC auth, IP target type) required.

---

## 19. DEPLOYMENTCONFIGS vs DEPLOYMENTS

**Q107. DeploymentConfig (DC) — legacy:**  
OpenShift's original Deployment abstraction. Supports ImageChange triggers (deploy when ImageStream updates), lifecycle hooks, custom strategies.

**Q108. Deployment vs DeploymentConfig:**  
- **Deployment** — Kubernetes-native, uses ReplicaSets.  
- **DeploymentConfig** — OpenShift, uses ReplicationControllers, image-change triggers, configurable deployment strategies (Rolling, Recreate, Custom).

**Q109. Is DeploymentConfig still recommended?**  
Red Hat marks DC as deprecated in favor of Deployment + Argo/Tekton for triggers. Many existing OpenShift workloads still use DC. **New work → Deployment.**

**Q110. DC features Deployment lacks:**  
- ImageStream trigger (auto-redeploy on new image).  
- Lifecycle hooks (pre/mid/post deployment).  
- Custom strategy pods.

**Q111. DC example:**  
```yaml
apiVersion: apps.openshift.io/v1
kind: DeploymentConfig
metadata: { name: my-app }
spec:
  replicas: 3
  triggers:
  - type: ImageChange
    imageChangeParams:
      automatic: true
      containerNames: [app]
      from: { kind: ImageStreamTag, name: my-app:latest }
  strategy:
    type: Rolling
    rollingParams: { maxUnavailable: 25%, maxSurge: 25% }
```

---

## 20. BUILDCONFIGS & IMAGESTREAMS

**Q112. BuildConfig — what is it?**  
OpenShift's in-cluster image build abstraction. Types:  
- **Source** (S2I — Source-to-Image) — inject source into runtime base image.  
- **Docker** — standard Dockerfile builds.  
- **Custom** — fully custom builder image.  
- **Pipeline** — Jenkins pipeline (deprecated; use Tekton).

**Q113. S2I (Source-to-Image) workflow:**  
1. `oc new-app <repo>` detects language.  
2. Creates BuildConfig with S2I builder.  
3. Build pod: pulls source + builder image + runs `assemble` script.  
4. Pushes resulting image to internal registry (ImageStream).  
5. DC trigger redeploys.

**Q114. ImageStream — what is it?**  
OpenShift abstraction over images. Tracks tags and references. Decouples image references in apps from actual registry + tag changes. ImageStreamTag can point to internal/external images.

**Q115. Why ImageStreams?**  
- Trigger redeployment on image change.  
- Abstract registry location (easy registry migration).  
- Central audit of image usage in cluster.  
- Mirror external images into internal registry.

**Q116. ImageStream example:**  
```yaml
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata: { name: my-app }
spec:
  tags:
  - name: latest
    from: { kind: DockerImage, name: quay.io/my-org/my-app:v1.2.3 }
    importPolicy: { scheduled: true }
```

**Q117. Internal image registry on ROSA:**  
Image Registry Operator manages — backing store is S3 bucket in your account (STS-assumed role). Registry route exposed: `default-route-openshift-image-registry.<apps-domain>`.

**Q118. Builds vs modern CI (Tekton):**  
BuildConfigs tie builds to the cluster. Modern pattern: Tekton Pipelines (OpenShift Pipelines) — more flexible, GitOps-friendly, decoupled from cluster namespaces. New work → Tekton.

---

## 21. OPENSHIFT OPERATORS & OPERATORHUB

**Q119. What is an Operator?**  
Custom controller + CRD packaging an application's install/upgrade/lifecycle knowledge. Introduced by CoreOS (now Red Hat); widespread in OCP.

**Q120. OperatorHub:**  
Web console / CLI catalog of Operators. Categories: Red Hat, Certified, Community, Marketplace. Install with one click (Subscription CR).

**Q121. OLM (Operator Lifecycle Manager):**  
Controller installing/upgrading operators via:  
- **CatalogSource** — index of operators.  
- **Subscription** — user's intent to install.  
- **InstallPlan** — approved install actions.  
- **ClusterServiceVersion (CSV)** — operator version manifest.

**Q122. Install mode — all namespaces vs own namespace:**  
- **AllNamespaces** — operator watches entire cluster (global operators).  
- **OwnNamespace** — operator scoped to single namespace (multi-tenant).  
Depends on operator's support.

**Q123. Subscription approval strategy:**  
- **Automatic** — upgrades applied as they come.  
- **Manual** — admin approves each upgrade (safer for prod).

**Q124. Example: install Cert-Manager operator:**  
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
spec:
  channel: stable-v1
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: Automatic
```

**Q125. Popular operators for ROSA workloads:**  
- OpenShift GitOps (ArgoCD).  
- OpenShift Pipelines (Tekton).  
- OpenShift Service Mesh (Istio).  
- OpenShift Serverless (Knative).  
- Logging Operator.  
- Monitoring Operator (platform).  
- Cert-Manager.  
- External Secrets Operator.  
- AWS Load Balancer Operator.  
- AWS Controllers for Kubernetes (ACK).

---

## 22. CLUSTER OPERATORS

**Q126. What are ClusterOperators?**  
Core platform operators managing cluster components: api-server, etcd, ingress, image-registry, monitoring, networking, etc. Visible via `oc get clusteroperators`.

**Q127. ClusterOperator conditions:**  
- `Available` — functional.  
- `Progressing` — in update.  
- `Degraded` — unhealthy.  
- `Upgradeable` — can be upgraded.  
First place to check during incidents.

**Q128. Platform operators on ROSA — managed by whom?**  
Red Hat SRE. Customers can inspect but not modify. Some configurations customer-controllable (e.g., image-registry storage).

**Q129. If ClusterOperator shows Degraded, what do?**  
Check `oc describe clusteroperator <name>` → conditions message. For ROSA, file support ticket — SRE resolves. For self-managed OCP, follow troubleshooting runbook.

---

## 23. MACHINESETS & MACHINE API

**Q130. Machine API:**  
OpenShift-specific controller managing cluster's EC2 nodes via `Machine`, `MachineSet`, `MachineAutoscaler` CRDs. Similar to AWS ASG but K8s-native.

**Q131. MachineSet — what is it?**  
Desired-state CRD for a group of nodes (similar to ReplicaSet for pods). One MachineSet per AZ typically. Controls instance type, AMI, labels, taints.

**Q132. MachineSet example fields:**  
- `spec.replicas` — desired count.  
- `spec.template.spec.providerSpec` — AWS-specific (instance type, AMI, IAM profile, subnet, SG).  
- `spec.template.spec.labels/taints` — applied to nodes.  
- `spec.template.spec.metadata` — annotations.

**Q133. Scaling nodes:**  
```bash
oc scale --replicas=5 machineset/<name> -n openshift-machine-api
# or via rosa CLI
rosa edit machinepool -c <cluster> <pool> --replicas 5
```

**Q134. MachinePool — ROSA abstraction:**  
Higher-level concept; managed via rosa CLI/OCM. Under the hood creates MachineSets. Use MachinePool for day-2 changes (preferred on ROSA).

**Q135. Adding spot instances on ROSA:**  
```bash
rosa create machinepool \
  --cluster my-cluster \
  --name spot-workers \
  --use-spot-instances \
  --replicas 5 \
  --instance-type m5.large \
  --labels environment=spot \
  --taints spot=true:NoSchedule
```

**Q136. MachineHealthCheck:**  
CRD for automatic node remediation. If node unhealthy for X minutes → MachineSet recreates. Essential for self-healing.

---

## 24. MACHINECONFIG & MACHINECONFIGPOOLS

**Q137. MachineConfig — what it does:**  
Declaratively manages RHCOS configuration (systemd units, files, kernel args) via ignition. Applied by Machine Config Operator (MCO).

**Q138. MachineConfigPool:**  
Groups of nodes with same MachineConfig (default: master + worker pools). Custom pools for infra nodes or specialized workloads.

**Q139. How updates apply:**  
1. New MachineConfig created / merged.  
2. MCO cordons + drains node.  
3. Reboots with new config.  
4. Node back online.  
5. Next node.  
Rolling, one at a time by default. Respects PDBs.

**Q140. Example — custom kernel parameter:**  
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 50-my-kernelarg
  labels: { machineconfiguration.openshift.io/role: worker }
spec:
  kernelArguments:
  - foo=bar
```

**Q141. ROSA restrictions on MachineConfig:**  
Customers can create MachineConfigs on `worker` pool (+ custom pools). Master pool protected. Avoid changes that conflict with SRE-managed settings.

**Q142. MachineConfig debugging:**  
- `oc get mcp` — pool status.  
- `oc describe node` — annotations show applied config.  
- `oc logs -n openshift-machine-config-operator <pod>` — MCO logs.  
- Check drain/reboot progress for stuck updates.

---

## 25. CLUSTER AUTOSCALING (OpenShift specific)

**Q143. OpenShift autoscaler — how it differs from upstream:**  
OpenShift Cluster Autoscaler uses MachineAutoscaler CRD (thin wrapper around MachineSets). CRDs: `ClusterAutoscaler` (global config) + `MachineAutoscaler` (per-MachineSet).

**Q144. MachineAutoscaler example:**  
```yaml
apiVersion: autoscaling.openshift.io/v1beta1
kind: MachineAutoscaler
metadata:
  name: worker-us-east-1a
  namespace: openshift-machine-api
spec:
  minReplicas: 2
  maxReplicaas: 20
  scaleTargetRef:
    apiVersion: machine.openshift.io/v1beta1
    kind: MachineSet
    name: cluster-name-worker-us-east-1a
```

**Q145. ClusterAutoscaler config options:**  
- Scale-down delays.  
- Pod priority cutoff (ignore below).  
- Max nodes total.  
- GPU resource limits.  
- `--balance-similar-node-groups`.

**Q146. HPA + ClusterAutoscaler cooperation:**  
HPA scales pods → if pending, MachineAutoscaler scales nodes via MachineSet → if nodes unused, scaled down. Same pattern as EKS + CA.

**Q147. Karpenter on ROSA?**  
Karpenter is Kubernetes-native; can run on ROSA but not officially supported/integrated. Red Hat recommends MachineAutoscaler. Some customers experiment with Karpenter for faster scaling.

---

## 26. STORAGE (EBS / EFS CSI ON ROSA)

**Q148. Default StorageClass on ROSA:**  
`gp3-csi` (EBS CSI) — default. Older clusters may have `gp2-csi`. Ingress operator uses this for PVCs.

**Q149. CSI drivers pre-installed:**  
- AWS EBS CSI (default).  
- AWS EFS CSI (operator — install separately if needed).  
- FSx CSI (via operator).  
- S3 CSI (preview).

**Q150. Install AWS EFS CSI Driver operator:**  
```bash
oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata: { name: aws-efs-csi-driver-operator, namespace: openshift-cluster-csi-drivers }
spec: { channel: stable, installPlanApproval: Automatic, name: aws-efs-csi-driver-operator, source: redhat-operators, sourceNamespace: openshift-marketplace }
EOF
```
Then create `ClusterCSIDriver` CR.

**Q151. EFS PVC example for ROSA:**  
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: { name: efs-sc }
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-xxx
  directoryPerms: "700"
```

**Q152. Registry storage on ROSA:**  
Internal image registry uses S3 bucket provisioned at install (STS-assumed role). Region-specific, encrypted.

**Q153. Backup volumes on ROSA:**  
Options: OpenShift API for Data Protection (OADP — Velero-based), EBS snapshots via CSI `VolumeSnapshot`, AWS Backup.

---

## 27. MONITORING STACK

**Q154. Default monitoring stack on ROSA:**  
Prometheus (cluster monitoring stack) + Alertmanager + Grafana (deprecated in console) + Thanos (long-term storage) + Telemeter (metrics to Red Hat for support).

**Q155. Customer's view of platform metrics:**  
Access via OpenShift Console → Observe → Metrics or PromQL. Platform metrics read-only. Some customer configuration (retention, remote write) allowed.

**Q156. User Workload Monitoring (UWM):**  
Separate Prometheus for customer workloads (cluster monitoring Prometheus isolated). Enable via ConfigMap in `openshift-monitoring`:  
```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: cluster-monitoring-config, namespace: openshift-monitoring }
data:
  config.yaml: |
    enableUserWorkload: true
```

**Q157. ServiceMonitor / PodMonitor:**  
CRDs (same as upstream Prometheus Operator) for defining scrape targets. Workloads expose `/metrics` endpoint; ServiceMonitor tells Prometheus how to scrape.

**Q158. Alerting:**  
AlertmanagerConfig CRD or central AlertManager config. Route to Slack, PagerDuty, email, webhook. Platform-level alerts preconfigured.

**Q159. Integrate with Amazon Managed Prometheus (AMP):**  
Configure remote-write on platform Prometheus to AMP for long-term storage / cross-cluster querying. Requires IAM role.

**Q160. Thanos vs Prometheus retention:**  
Platform Prometheus: 15d default. Thanos ruler/query federates for longer. Configure AMP remote-write for enterprise retention.

---

## 28. LOGGING STACK

**Q161. Default logging on ROSA:**  
**Not installed by default** — customer installs Logging Operator (CLO). Options:  
- **Vector/Loki** (modern stack — Red Hat's direction).  
- **Fluentd/Elasticsearch** (legacy — being removed in OCP 5.x).  
- **Forward to external (CloudWatch, Splunk, Datadog)** via ClusterLogForwarder.

**Q162. Why Loki over Elasticsearch?**  
- Label-based index (cheaper storage).  
- Object store backend (S3 on AWS).  
- Designed for log querying at scale.  
- Elasticsearch stack removed from OCP Logging 6.

**Q163. ClusterLogForwarder:**  
CRD defining input pipelines + outputs. Flexible: app/audit/infra logs → multiple destinations.

**Q164. Example: forward app logs to CloudWatch + audit to S3:**  
```yaml
apiVersion: logging.openshift.io/v1
kind: ClusterLogForwarder
metadata: { name: instance, namespace: openshift-logging }
spec:
  outputs:
  - name: cw
    type: cloudwatch
    cloudwatch: { groupBy: logType, region: us-east-1 }
  - name: s3-audit
    type: s3
    s3: { bucket: my-audit-bucket, region: us-east-1 }
  pipelines:
  - name: app-to-cw
    inputRefs: [application]
    outputRefs: [cw]
  - name: audit-to-s3
    inputRefs: [audit]
    outputRefs: [s3-audit]
```

**Q165. Audit logs on ROSA:**  
API server audit logs generated by platform; forwarded via ClusterLogForwarder (customer-configurable). Critical for compliance.

---

## 29. OPENSHIFT GITOPS (ArgoCD)

**Q166. What is OpenShift GitOps?**  
Red Hat-supported ArgoCD operator. Installs ArgoCD with OCP integrations (OAuth SSO, Dex IDP, RBAC binding).

**Q167. Install via operator:**  
OperatorHub → "Red Hat OpenShift GitOps" → Install. Creates default ArgoCD instance in `openshift-gitops` namespace.

**Q168. Multiple ArgoCD instances:**  
Create additional `ArgoCD` CR for per-team or per-env instances. Common pattern: central platform ArgoCD + tenant ArgoCDs.

**Q169. OpenShift GitOps vs vanilla ArgoCD:**  
Nearly identical UX; differences:  
- Pre-integrated OAuth with OpenShift IDPs.  
- Default RBAC tuned for OpenShift resources (Routes, BuildConfigs).  
- SRE-supported (enterprise).  
- Health checks for OpenShift CRDs.

**Q170. ApplicationSet for fleet:**  
Deploy same app to many clusters/namespaces via Generators (list, cluster, git). Great for multi-cluster ROSA fleet management.

**Q171. Syncing cluster-scoped resources:**  
ArgoCD needs cluster-admin to manage ClusterRoles, SCCs, etc. Usually platform ArgoCD has cluster-admin; tenant ArgoCDs restricted to namespaces.

---

## 30. OPENSHIFT PIPELINES (Tekton)

**Q172. What is OpenShift Pipelines?**  
Red Hat-packaged Tekton — cloud-native CI/CD on Kubernetes. Install via operator.

**Q173. Core Tekton concepts:**  
- **Task** — reusable unit (clone git, run tests).  
- **Pipeline** — ordered workflow of Tasks.  
- **PipelineRun** — execution instance.  
- **TaskRun** — single Task execution.  
- **Workspaces** — shared volumes between tasks.  
- **Triggers** — webhook → pipeline.

**Q174. Why Tekton over Jenkins on ROSA?**  
- Kubernetes-native (pods as build agents, scale with cluster).  
- No persistent controller (ephemeral).  
- YAML-first (GitOps-friendly).  
- Tightly integrated with OpenShift (Pipelines as Code, GitHub App).

**Q175. Tekton vs Jenkins X vs Argo Workflows:**  
- **Tekton** — industry-standard CI/CD primitives.  
- **Jenkins X** — deprecated heavy abstraction.  
- **Argo Workflows** — DAG-based for data pipelines.  
ROSA recommendation: Tekton via OpenShift Pipelines.

**Q176. Pipelines as Code (PaC):**  
Git-based pipeline definitions. GitHub/GitLab webhook triggers PipelineRun per commit. Config in `.tekton/` directory of repo.

---

## 31. OPENSHIFT SERVICE MESH (Istio based)

**Q177. What is OpenShift Service Mesh?**  
Red Hat-packaged Istio (via Maistra project for multi-tenancy). Install via operator. Managed control plane + data plane (sidecars).

**Q178. Differences from upstream Istio:**  
- Supports multiple control planes in cluster (multi-tenant).  
- Integrates with OpenShift SCCs (sidecar needs specific SCC).  
- Red Hat SRE support option.  
- Opinionated defaults.

**Q179. Installing Service Mesh:**  
1. Install OpenShift Service Mesh Operator + Kiali Operator + Jaeger Operator.  
2. Create `ServiceMeshControlPlane` CR in a namespace.  
3. Create `ServiceMeshMemberRoll` listing namespaces joining mesh.  
4. Apply sidecar injection annotation.

**Q180. Kiali + Jaeger:**  
- **Kiali** — visual service mesh dashboard (topology, traffic flow).  
- **Jaeger** — distributed tracing.  
Bundled with Service Mesh install.

**Q181. Istio ambient mesh (sidecarless) — available on ROSA?**  
Upstream Istio ambient is evolving; Red Hat Service Mesh 3 roadmap aligns. Check current version's support.

---

## 32. OPENSHIFT SERVERLESS (Knative)

**Q182. What is OpenShift Serverless?**  
Red Hat-packaged Knative — Kubernetes-native serverless framework (scale-to-zero, event-driven). Install via operator.

**Q183. Knative components:**  
- **Serving** — request-driven autoscaling, revisions, traffic splitting.  
- **Eventing** — event brokers, triggers, sources.  
- **Functions** — language SDK for serverless functions.

**Q184. When use Knative on ROSA vs Lambda?**  
- Knative — multi-cloud portability, K8s ecosystem, no Lambda limits (15 min timeout, 10 GB memory).  
- Lambda — AWS-native, true serverless (no K8s cost baseline), extensive event source integration.

**Q185. Knative Service scale-to-zero:**  
Inactive service scales replicas to 0 after idle timeout. Cold start on request. Configurable `autoscaling.knative.dev/minScale`, `maxScale` annotations.

---

## 33. UPGRADES & LIFECYCLE

**Q186. ROSA upgrade channels:**  
- **Stable** — well-tested.  
- **Fast** — newer, faster access to versions.  
- **Candidate** — pre-release.  
Default: stable. Choose per cluster based on risk tolerance.

**Q187. Upgrade process:**  
1. Check available versions: `rosa list upgrades -c <cluster>`.  
2. Schedule: `rosa upgrade cluster -c <cluster> --version 4.16.x`.  
3. Red Hat SRE orchestrates control plane upgrade.  
4. MachineConfigPools update nodes rolling (1 at a time by default).  
5. All operators converge to new version.

**Q188. Minor vs patch upgrades:**  
- **Patch** (z-stream) — 4.15.5 → 4.15.6 — bug fixes, no API changes.  
- **Minor** (y-stream) — 4.15 → 4.16 — new features, possible API deprecations.  
Apply patches aggressively; minor with care + staging tests.

**Q189. Upgrade windows:**  
Configure preferred time for SRE-triggered upgrades (e.g., maintenance window). EUS (Extended Update Support) releases: longer support window, skip minor versions allowed.

**Q190. EUS (Extended Update Support):**  
Even-numbered minor versions (4.14, 4.16, 4.18) get 24 months of support. Upgrade 4.14 → 4.15 → 4.16 (intermediate skipped in console workflow for EUS users).

**Q191. Stopping an in-progress upgrade:**  
Limited — control plane upgrade once started difficult to abort. Worker pool upgrade (via MachineConfigPool pause) can be halted.

**Q192. Pre-upgrade checks:**  
- `oc adm upgrade` — check for errors.  
- Deprecated APIs (kube-no-trouble / kubent).  
- Operator compatibility.  
- Cluster health (`oc get co`).  
- Custom resources (webhooks functioning).

---

## 34. BACKUP & DISASTER RECOVERY (OADP)

**Q193. What is OADP?**  
**OpenShift API for Data Protection** — operator wrapping Velero + restic/kopia. Backup and restore of cluster workloads + PVs.

**Q194. OADP components:**  
- Velero core.  
- Data mover (CSI snapshot or file-level backup via kopia/restic).  
- Plugin for OpenShift (handles OpenShift-specific resources — BuildConfigs, ImageStreams, SCCs).

**Q195. Backup a namespace:**  
```yaml
apiVersion: velero.io/v1
kind: Backup
metadata: { name: my-app-backup, namespace: openshift-adp }
spec:
  includedNamespaces: [my-app]
  storageLocation: aws
  defaultVolumesToFsBackup: true
```

**Q196. Restore patterns:**  
- Same cluster (operational mistake recovery).  
- Different cluster (migration, DR).  
- Selective (namespace / label).

**Q197. ROSA cluster-level DR options:**  
- ROSA doesn't provide native cluster-level backup (control plane is SRE-managed).  
- Customer responsibility: workload + data via OADP.  
- For cluster-level failure: rebuild new cluster + OADP restore.

**Q198. Multi-region DR:**  
Run standby ROSA cluster in DR region. GitOps (ArgoCD) deploys apps. OADP restores stateful data (PVs from S3 backup). Route 53 failover.

---

## 35. COMPLIANCE & SECURITY OPERATORS

**Q199. Compliance Operator:**  
Scans cluster against security profiles (CIS, NIST, PCI-DSS). Based on OpenSCAP. Reports deviations with remediation.

**Q200. ComplianceSuite / ScanSetting CRDs:**  
Schedule periodic scans → ComplianceCheckResult CRs → optionally apply auto-remediation.

**Q201. File Integrity Operator:**  
Monitors file changes on nodes (RHCOS). Detects tampering. AIDE-based.

**Q202. Security Profiles Operator:**  
Manages SELinux / seccomp profiles at cluster level. Apply to pods via annotations.

**Q203. RHACS (Red Hat Advanced Cluster Security) — previously StackRox:**  
Separate product (add-on). Comprehensive K8s security platform: vulnerability scanning, runtime detection, network graph, CIS compliance, policy engine. Often adopted alongside ROSA for enterprise security.

**Q204. Trusted Artifact Signer / Sigstore:**  
OCP 4.15+ supports cosign-signed image verification via ClusterImagePolicy CRD. Enforce only signed images.

---

## 36. COST OPTIMIZATION ON ROSA

**Q205. Top ROSA cost levers:**  
1. **Right-size compute** — check MachinePool instance types; Graviton (arm64) support.  
2. **Spot instances** for non-critical workloads (MachinePool with `--use-spot-instances`).  
3. **Autoscaling** — MachineAutoscaler to scale down at night.  
4. **HCP over Classic** — eliminate master/infra costs.  
5. **Reserved Instances / Compute Savings Plans** — for steady baseline.  
6. **EBS gp3 over gp2** — cheaper + better performance.  
7. **Consolidate small clusters** — many HCP clusters only if isolation required.  
8. **Use internal registry with S3 Intelligent-Tiering**.  
9. **Logging via S3-backed Loki** over hot Elasticsearch.  
10. **ROSA monitoring retention tuning**.

**Q206. ROSA cost monitoring tools:**  
- AWS Cost Explorer tagged by `red-hat-clustertype`, `red-hat-managed`.  
- Kubecost / OpenCost (install via operator).  
- Subscription Watch (Red Hat — CPU tracking).

**Q207. Shut down cluster at night:**  
For dev clusters, `rosa edit machinepool --replicas 0` scales workers to 0. Classic control plane still running (you pay for masters/infra). HCP — no control plane cost. Consider delete/recreate for pure dev.

---

## 37. SCENARIO-BASED QUESTIONS

**Q208. Design: enterprise ROSA platform for 30 dev teams, each their own namespace.**  
- Classic ROSA cluster (3 AZ, multi-tenant).  
- `dedicated-admin` for team leads on their projects.  
- ResourceQuotas + LimitRanges per project.  
- NetworkPolicies isolating namespaces.  
- OpenShift GitOps (central ArgoCD) deploys per team.  
- OpenShift Pipelines per team namespace.  
- Project template auto-populates RBAC, SCC bindings.  
- Compliance Operator scans weekly.  
- Logging to CloudWatch + metrics to AMP.  
- Kubecost for chargeback.

**Q209. Migrate from self-managed OCP on VMware to ROSA — approach.**  
1. Capacity planning → right-size ROSA (node count, machine types).  
2. STS prerequisites + BYOVPC design.  
3. Create ROSA cluster (start with Classic or HCP).  
4. Install parity operators (GitOps, Pipelines, Cert-Manager, Service Mesh).  
5. Replicate images to internal registry (mirror).  
6. Migrate manifests via GitOps — selectively test per app.  
7. Move state: databases via replication; volumes via OADP/Velero.  
8. Cut traffic via DNS.  
9. Validate + decommission VMware side.

**Q210. SCC conflict — app team's Helm chart fails on ROSA but works on EKS.**  
- Default `restricted-v2` SCC rejects root containers.  
- Options:  
  1. Fix image to run as non-root (preferred).  
  2. Grant `anyuid` SCC to app's ServiceAccount (scoped; don't cluster-wide).  
  3. Use custom SCC with narrow `runAsUser` range.  
- Document for other teams; create "ROSA migration checklist."

**Q211. Compliance audit requires immutable logs for 7 years.**  
- ClusterLogForwarder → S3 with Object Lock (compliance mode, 7y).  
- Cross-region S3 replication.  
- KMS encryption CMK.  
- CloudTrail Lake for API audit.  
- Compliance Operator scans for PCI/CIS.  
- File Integrity Operator for node-level tampering.

**Q212. Dev team wants Spot nodes for CI workloads — safely.**  
- Create Spot MachinePool with taint `spot=true:NoSchedule`.  
- Tolerations on CI pods only.  
- PDB not critical (CI tolerates restarts).  
- Node draining handler for spot interruption.  
- Label for cost allocation.

**Q213. HCP cost-effective fleet: 50 small dev clusters.**  
- HCP mode: minimal worker nodes per cluster (2 × m5.large).  
- Shared services account (monitoring, logging central).  
- Infrastructure as Code (Terraform) for reproducible clusters.  
- ApplicationSet deploys baseline apps to all.  
- Nightly shutdown via rosa CLI + cron (scale workers to 0).

**Q214. Tenant isolation: one customer's data must never touch another's.**  
- Separate ROSA clusters per tenant (strongest).  
- OR single cluster with strict NetworkPolicies, separate SCs (encryption per tenant), separate registries per namespace, AdminNetworkPolicy enforcing baseline.  
- IAM at AWS layer — separate S3 buckets / KMS keys per tenant via operator roles.

**Q215. Critical app's pod stuck in CrashLoopBackOff after ROSA 4.15→4.16 upgrade.**  
- Check deprecated API removed in 4.16 (e.g., specific CRD version).  
- `oc adm upgrade status` for ClusterOperators.  
- Operator version compatibility with new OCP.  
- Rollback not straightforward — file Red Hat support ticket.

**Q216. IAM roles for operator drifted; OIDC broken.**  
- `rosa verify permissions` shows missing perms.  
- `rosa delete operator-roles` + recreate.  
- Don't manually edit STS roles (Terraform / rosa CLI only).

**Q217. Egress IP for pods to access 3rd-party API with IP allowlist.**  
- EgressIP feature (OVN).  
- Assign static IP from subnet to namespace.  
- Configure 3rd party to allowlist that IP.  
- Multi-AZ considerations (failover IP).

**Q218. Route not resolving externally.**  
- DNS CNAME for `*.apps.<cluster>.<domain>` points to Route 53 / ELB?  
- IngressController healthy?  
- Route status conditions (`oc get route <name> -o yaml`).  
- TLS cert valid?

**Q219. Want to run ROSA on Outposts (edge).**  
ROSA on Outposts limited availability; check Red Hat/AWS roadmap. For on-prem OpenShift at edge: OpenShift 4 on bare metal or single-node OpenShift (SNO).

---

## 38. TROUBLESHOOTING QUESTIONS

**Q220. `oc login` fails "Unauthorized":**  
- Wrong IDP selected.  
- Token expired.  
- `kubeadmin` removed — use IDP user.  
- Network issue to API.

**Q221. `rosa create cluster` failed with "permission denied":**  
- STS roles missing actions.  
- `rosa verify permissions` check.  
- Recreate account roles (`rosa delete account-roles` + create).

**Q222. Pods stuck in ContainerCreating — "failed to create pod sandbox":**  
- CNI (OVN) pods unhealthy.  
- Node out of IPs (VPC subnet exhausted).  
- PSP/SCC rejection (check events).  
- CRI-O storage driver issue.

**Q223. Pod stuck Pending with "node(s) had untolerated taint":**  
- Node has taint (e.g., `node-role.kubernetes.io/infra`).  
- Add toleration to pod spec.  
- Or schedule to worker nodes (selectors).

**Q224. Route returns 503 Service Unavailable:**  
- Backend service has no endpoints (pods unhealthy).  
- Route pointing to wrong service/port.  
- Router health check failing.  
- Selector mismatch.

**Q225. Build fails with "imagestream not found":**  
- ImageStream ref wrong namespace.  
- BuildConfig permissions missing (SA can't read IS).  
- ImageStream import failed (registry auth / network).

**Q226. "UPGRADEABLE: False" on a ClusterOperator:**  
- Operator blocking upgrade (usually good — prevents data loss).  
- Check operator logs for specific precondition not met.  
- For ROSA: contact Red Hat support.

**Q227. MachineConfig stuck applying — pool degraded:**  
- Drain failing due to PDB.  
- Node failed to reboot.  
- `oc describe mcp worker` for Degraded reason.  
- Check node events, kubelet logs.

**Q228. `oc adm top nodes` returns "error":**  
- Metrics server / Prometheus unavailable.  
- Node not reporting metrics.  
- ClusterOperator `monitoring` degraded.

**Q229. Image pull from internal registry fails:**  
- Image puller role missing on SA (`system:image-puller` binding).  
- Image does not exist.  
- Internal registry route unreachable from node.

**Q230. Certificate expired on Route:**  
- Custom cert expiration — rotate via ACM/cert-manager.  
- Platform ingress cert — SRE-managed; file ticket.

**Q231. `oc new-project` fails "you may not request a new project":**  
- User lacks `self-provisioner` role.  
- Project template validation failure.  
- OAuth token refresh needed.

**Q232. CoreDNS / NodeLocalDNS returning SERVFAIL:**  
- DNS operator degraded.  
- Egress firewall blocking DNS upstream.  
- ConfigMap corrupted.

**Q233. Prometheus OOMKilled:**  
- Cardinality explosion (one workload emitting too many series).  
- Query from UI too expensive.  
- Increase memory via ClusterMonitoringConfig.

---

## 39. REAL-TIME PRODUCTION ISSUES

**Q234. Cluster upgrade stalled overnight, status Progressing.**  
- Check ClusterOperator conditions.  
- Drain stuck on a PDB-protected workload.  
- File Red Hat ticket with `oc adm must-gather` output.

**Q235. New ingress cert not applied after update.**  
- IngressController CR not updated with new secret ref.  
- Router pods not rolled.  
- Validate via `openssl s_client`.

**Q236. Team's Helm chart creates ClusterRoles; dedicated-admin insufficient.**  
- Grant `cluster-admin` temporarily OR refactor chart to not create cluster-scoped resources.  
- Use ArgoCD with cluster-admin (platform-managed) to install.

**Q237. Etcd performance slow (reported by Red Hat SRE) on Classic cluster.**  
- Master nodes undersized.  
- Scale up master instance type (rosa edit).  
- Archive large Events / reduce watch count.

**Q238. Custom Operator conflicts with platform operator.**  
- Both installing same CRD.  
- OLM catches; you see InstallPlan error.  
- Uninstall one; choose carefully.

**Q239. Egress from pods failing randomly to internet.**  
- NAT Gateway port exhaustion.  
- Multiple NAT GWs (one per AZ) help.  
- Investigate connection patterns; pooling.

**Q240. Developer accidentally deleted `openshift-monitoring` namespace.**  
- Platform namespaces protected by admission webhooks (shouldn't allow deletion).  
- If somehow deleted, SRE recovery required.  
- SCP / admission control to prevent cluster-admin from doing this.

**Q241. IAM operator role permissions drifted (manual edit).**  
- `rosa verify permissions` fails.  
- Cluster operators fail AWS API calls.  
- Recreate via `rosa create operator-roles`.

**Q242. Cost spike due to log volume.**  
- Vector/Loki ingestion cost.  
- Container logging at DEBUG.  
- CloudWatch Logs egress.  
- Filter at source; sample; increase retention aggressiveness.

**Q243. Prometheus remote-write failures to AMP.**  
- IAM role for operator missing `aps:RemoteWrite`.  
- Signing issue — check AMP config.  
- Backpressure from AMP.

**Q244. OADP backup succeeded but restore fails.**  
- Cluster-scoped resources (CRDs, SCCs) missing at destination.  
- Version skew between source and target cluster.  
- PV / snapshot region mismatch.

---

## 40. DEVOPS / IaC FOCUSED ROSA QUESTIONS

**Q245. Terraform for ROSA — providers used:**  
- `rhcs` (HashiCorp-maintained) — cluster, machinepools, IDPs.  
- `aws` — VPC, subnets, IAM STS roles.  
- `helm`/`kubernetes` — in-cluster resources (preferably via ArgoCD).

**Q246. Example Terraform snippet for ROSA HCP:**  
```hcl
resource "rhcs_cluster_rosa_hcp" "cluster" {
  name           = "my-hcp"
  cloud_region   = "us-east-1"
  aws_subnet_ids = [aws_subnet.public.id, aws_subnet.private.id]
  sts = {
    role_arn         = module.iam.installer_role_arn
    support_role_arn = module.iam.support_role_arn
    # ... operator_role_prefix etc
  }
  compute_machine_type = "m5.xlarge"
  replicas             = 2
}
```

**Q247. GitHub Actions for ROSA deploy:**  
- OIDC federation → IAM role.  
- `rosa` CLI installed in workflow.  
- `rosa login --token` with secret.  
- Deploy manifests via `oc apply` OR better via ArgoCD sync trigger.

**Q248. Cluster-as-Code patterns:**  
- Terraform module per cluster.  
- Variables for environment (dev/staging/prod).  
- CI promotes through environments.  
- Cluster baseline: ArgoCD self-managed from Git.  
- "App of Apps" pattern bootstraps the cluster.

**Q249. Bootstrap ArgoCD on new ROSA cluster:**  
1. Terraform creates ROSA cluster.  
2. Post-create provisioner: install OpenShift GitOps operator.  
3. Create ArgoCD CR.  
4. Bootstrap Git repo configured as root App.  
5. ArgoCD manages everything else.

**Q250. Managing IDP config in IaC:**  
```hcl
resource "rhcs_identity_provider" "okta" {
  cluster = rhcs_cluster_rosa_classic.cluster.id
  name    = "Okta"
  openid  = {
    client_id     = var.okta_client_id
    client_secret = var.okta_client_secret
    issuer        = "https://<okta-domain>"
    claims        = { email = ["email"], name = ["name"], username = ["preferred_username"] }
  }
}
```

**Q251. Drift detection on ROSA IaC:**  
- `terraform plan` scheduled.  
- ArgoCD for in-cluster drift.  
- CloudTrail for AWS-level changes (IAM, VPC).  
- Policy-as-code (Sentinel / OPA) blocks bad changes in PR.

**Q252. Secrets in ROSA IaC:**  
- Vault / Secrets Manager referenced by External Secrets Operator.  
- Don't commit secrets to Git.  
- Terraform: `sensitive = true` + encrypted remote state (S3 + KMS).  
- Sealed Secrets for GitOps if External Secrets not available.

**Q253. Multi-cluster fleet management:**  
- ArgoCD ApplicationSet.  
- Red Hat Advanced Cluster Management (ACM/RHACM) — separate add-on product.  
- Hub cluster pattern (one ROSA manages policies across fleet).

**Q254. Testing ROSA manifests:**  
- `oc diff -f manifest.yaml` — show changes before apply.  
- `conftest` with OPA Rego — policy tests.  
- Dev cluster environment mirrors prod CRDs.  
- `kyverno test` for policy.

**Q255. Automating upgrades via IaC:**  
```bash
rosa upgrade cluster -c my-cluster --version 4.16.10 --schedule "0 2 * * Sun"
```
Or Terraform `version` change triggers upgrade. Combine with pre-checks (kube-no-trouble scan).

---

## BONUS: 20 RAPID-FIRE ONE-LINERS

1. What is RHCOS? → Red Hat CoreOS, immutable container OS.  
2. Default SCC? → `restricted-v2`.  
3. Default CNI? → OVN-Kubernetes.  
4. ROSA SLA? → 99.95%.  
5. Control plane location in HCP? → Red Hat-managed AWS account.  
6. Minimum nodes for Classic multi-AZ? → 9 (3 masters + 3 infra + 3 workers).  
7. Minimum for HCP? → 2 workers.  
8. ROSA CLI binary? → `rosa`.  
9. OpenShift CLI binary? → `oc`.  
10. Internal registry backing store? → S3.  
11. `oc new-project` vs `kubectl create ns`? → `oc new-project` creates Project (+ metadata).  
12. Route vs Ingress? → Route is OpenShift L7 abstraction pre-dating Ingress.  
13. `system:admin` — how to access? → Limited on ROSA; via cluster-admin granted user.  
14. ClusterOperators total count typical? → 30+ depending on version.  
15. Default Pod CIDR? → 10.128.0.0/14.  
16. Default Service CIDR? → 172.30.0.0/16.  
17. Can install Karpenter on ROSA? → Technically yes, not officially supported.  
18. ArgoCD on ROSA flavor? → OpenShift GitOps (Red Hat-packaged).  
19. Knative on ROSA? → OpenShift Serverless.  
20. Can ROSA use Graviton? → Yes, select ARM64 MachinePool instance types.

---

## COMMON "WHY" FOLLOW-UPS

- Why ROSA vs self-managed OCP?  
- Why STS over IAM user mode?  
- Why SCC strict and not PSA-only?  
- Why Routes in addition to Ingress?  
- Why DeploymentConfig being deprecated?  
- Why BuildConfigs losing favor to Tekton?  
- Why HCP is cheaper than Classic?  
- Why OpenShift opinionated vs EKS unopinionated?  
- Why Red Hat packages operators (GitOps, Pipelines, ServiceMesh) vs pure upstream?  
- Why dedicated-admin over cluster-admin by default?

---

## MEE PROFILE KI INTERVIEW STRATEGY

Mee **ROSA + K8s + OpenShift + AWS DevOps** combo is rare in Indian market. Ee question bank preparation chesinaka, interviews lo focus cheyandi:

1. **Storytelling** — "we migrated from X to ROSA because..." — real experience narrative.  
2. **Classic vs HCP opinion** — interviewer ki mee architectural thinking show avuthundi.  
3. **SCC deep knowledge** — OpenShift-specific, EKS folks ki ledu.  
4. **rosa CLI + oc CLI fluency** — live demo asked sometimes.  
5. **Red Hat ecosystem** — GitOps, Pipelines, Service Mesh, Serverless — operator pattern.  
6. **Integration** — ROSA + ArgoCD + Terraform + Jenkins (mee existing stack).  
7. **Compliance angle** — Red Hat partners lo (Wipro, TCS Red Hat practice, Accenture Red Hat) huge.  

---

## SUGGESTED 5-WEEK PREP PLAN (given mee ROSA background)

1. **Week 1** — ROSA fundamentals, architectures, STS, CLI (Q1–Q54).  
2. **Week 2** — Cluster creation, networking, VPC, IDPs, Projects, SCCs (Q55–Q98).  
3. **Week 3** — Routes, DC/BC/IS, Operators, MachineSets, MachineConfig (Q99–Q147).  
4. **Week 4** — Storage, Monitoring, Logging, GitOps, Pipelines, Service Mesh (Q148–Q205).  
5. **Week 5** — Scenarios, troubleshooting, production, DevOps (Q206–Q255).  
6. Lab: Create ROSA HCP cluster with BYOVPC + STS + OpenShift GitOps + sample app with Routes.  
7. Practice `rosa` + `oc` CLI until muscle memory.  
8. Always frame with: **managed service benefits, multi-tenancy, compliance, cost, OpenShift opinionated value**.

---

*End of Document – 255+ unique questions across all ROSA/OpenShift modules. No duplication with EC2/VPC/IAM/S3/EKS/Lambda/Route 53/CloudWatch/CloudTrail banks.*
