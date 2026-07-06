# AWS EKS – Complete Interview Question Bank
**Target Audience:** 3–4+ years experience DevOps / Cloud / SRE / Platform engineers (with K8s / OpenShift / ROSA background)  
**Scope:** All EKS modules + Real-time + Scenario-based + Troubleshooting  
**Source mix:** MNC interview patterns (TCS, Infosys, Wipro, Cognizant, Accenture, Deloitte, Capgemini, Amazon, Microsoft, IBM, HCL, LTIM, Mphasis, Red Hat, VMware) + LinkedIn shared questions + real production incidents.  
**Note:** No duplication with EC2 / VPC / IAM / S3 banks.

---

## TABLE OF CONTENTS
1. EKS Fundamentals
2. EKS Control Plane vs Data Plane
3. Cluster Creation & Configuration
4. Node Types (Managed / Self-managed / Fargate)
5. Networking — VPC CNI Deep Dive
6. Alternative CNIs (Calico, Cilium)
7. Pod Networking & Security Groups for Pods
8. Ingress Controllers & AWS Load Balancer Controller
9. Service Types in EKS
10. Storage in EKS (EBS CSI / EFS CSI / FSx CSI)
11. IRSA (IAM Roles for Service Accounts)
12. EKS Pod Identity
13. Authentication & Authorization (aws-auth / Access Entries)
14. EKS Add-ons
15. Cluster Upgrades
16. Auto Scaling — Cluster Autoscaler
17. Karpenter
18. HPA / VPA / KEDA
19. Observability (Container Insights, Logs, Metrics, Tracing)
20. Logging & Fluent Bit / FluentD
21. Service Mesh (App Mesh, Istio on EKS)
22. Secrets Management (Secrets Manager, External Secrets)
23. EKS Fargate Deep Dive
24. EKS on Outposts / Anywhere / Hybrid
25. Cost Optimization
26. Security & Compliance
27. Upgrades & Disaster Recovery
28. GitOps (ArgoCD / Flux) on EKS
29. CI/CD Integration
30. Scenario-Based Questions
31. Troubleshooting Questions
32. Real-time Production Issues
33. DevOps / IaC Focused EKS Questions

---

## 1. EKS FUNDAMENTALS

**Q1. What is EKS and how is it different from self-managed Kubernetes?**  
Managed Kubernetes control plane — AWS runs etcd, API server, scheduler, controller-manager across 3 AZs. You manage only data plane (nodes) and workloads. Eliminates 80% ops burden vs DIY (kubeadm/kops).

**Q2. EKS vs ECS vs ROSA — positioning:**  
- **EKS** — upstream Kubernetes, portability, huge ecosystem.  
- **ECS** — AWS-proprietary scheduler, tighter AWS integration, simpler learning curve.  
- **ROSA** — Red Hat OpenShift on AWS (jointly managed). OCP features (developer console, built-in CI, stricter security defaults). Your ROSA background helps understand EKS since both run Kubernetes but EKS is "bare" while ROSA gives OpenShift layer on top.

**Q3. EKS pricing model:**  
$0.10/hour per cluster (control plane) + EC2/Fargate node cost + EBS/EFS storage + data transfer. Extended Support adds ~$0.50/hour/cluster.

**Q4. Key EKS concepts for Kubernetes veterans:**  
- `kubectl` still primary tool.  
- `aws eks update-kubeconfig` fetches kubeconfig with IAM-based auth.  
- Service accounts use IRSA/Pod Identity for AWS API access.  
- No tiller (deprecated), Helm 3 standard.

**Q5. EKS supported Kubernetes versions and lifecycle:**  
AWS supports 4 minor versions typically (N, N-1, N-2, N-3). Standard support 14 months, then Extended Support up to 26 months at extra cost. Always upgrade before EOS to avoid security gaps.

---

## 2. EKS CONTROL PLANE vs DATA PLANE

**Q6. What's inside EKS control plane?**  
API server (min 2 instances across AZs), etcd (3 replicas across AZs), scheduler, controller manager, cloud controller manager. All managed by AWS — no SSH/logs visible (except via CloudWatch Logs enablement).

**Q7. Can you access the EKS control plane directly?**  
No SSH. You interact via Kubernetes API endpoint (public/private/both). Logs via CloudWatch (api, audit, authenticator, controllerManager, scheduler).

**Q8. EKS endpoint access modes:**  
- **Public** — Kubernetes API reachable from internet (allowlist via CIDR).  
- **Private** — Only from within VPC (via interface endpoint).  
- **Public + Private** — Both, common for workstation access + in-VPC nodes.

**Q9. What is the EKS API endpoint's source IP when nodes communicate?**  
When private access enabled, nodes talk to control plane via ENIs (`eks-cluster-sg`) in your VPC. Public mode routes over internet.

**Q10. Can you have multiple API server endpoints for HA?**  
AWS manages HA automatically (multiple API servers across AZs behind a single NLB-like endpoint). You just use the one DNS name.

---

## 3. CLUSTER CREATION & CONFIGURATION

**Q11. Tools to create EKS cluster:**  
- **eksctl** (most popular, YAML-driven)  
- **Terraform** (`terraform-aws-eks` module — industry standard)  
- **CloudFormation**  
- **CDK**  
- **AWS Console / CLI**  
- **Rancher, Pulumi** (third-party)

**Q12. What subnets does EKS require?**  
Minimum 2 subnets in 2 different AZs. Tags matter:  
- `kubernetes.io/cluster/<cluster-name>=shared|owned`  
- `kubernetes.io/role/elb=1` (public subnet for internet LBs)  
- `kubernetes.io/role/internal-elb=1` (private subnet for internal LBs)

**Q13. Can EKS span multiple regions?**  
No. One cluster = one region. Multi-region = multiple clusters with federation/service mesh.

**Q14. Control plane security group — what's its role?**  
ENIs injected into your VPC. The SG (`eks-cluster-sg-<name>-...`) allows port 443 from nodes, and node ports from control plane. Don't delete or heavily modify.

**Q15. Cluster encryption — what's encrypted?**  
KMS envelope encryption for **Kubernetes Secrets** at etcd level. Configure at creation (cannot change easily later). Bring-your-own CMK for compliance.

**Q16. Secrets encryption — etcd already encrypted at rest?**  
AWS encrypts etcd at rest by default with AWS-managed keys. Enabling envelope encryption adds **customer-managed KMS** double-encryption — required for some compliance (HIPAA, FedRAMP).

**Q17. Can you migrate existing Kubernetes workloads to EKS?**  
Yes — standard kubectl apply if manifests compatible. Tools: Velero for backup/restore, AWS Migration Hub, AWS App2Container. Watch for storage class changes, LB controller differences, CNI migration.

---

## 4. NODE TYPES

**Q18. Three node types on EKS — compare:**  
| | Managed Node Group | Self-Managed | Fargate |
|---|---|---|---|
| EC2 managed by | AWS | You | N/A (serverless) |
| AMI | AWS-optimized, auto-update | Any | N/A |
| Scaling | ASG + EKS API | ASG only | Per-pod |
| Upgrade | One-click rolling | Manual | Automatic |
| Patching | AWS releases AMI | You | AWS |
| OS options | AL2/AL2023/Bottlerocket/Ubuntu/Windows | Any | AL2 (managed) |
| Spot support | Yes (mixed) | Yes | No (On-Demand only) |
| GPU | Yes | Yes | No |
| Pricing | EC2 + EBS | EC2 + EBS | Per pod vCPU/GB-hour |
| Custom user-data | Launch Template | Yes | No |

**Q19. Managed Node Group — how upgrades work:**  
AWS respects PDBs, drains nodes, launches new nodes with new AMI. Max unavailable controls parallelism. Can force if PDB blocks.

**Q20. Bottlerocket vs Amazon Linux 2 vs AL2023:**  
- **Bottlerocket** — container-optimized, minimal OS, atomic updates, security-focused (API-driven, no SSH by default). Ideal for production.  
- **AL2** — general-purpose, Docker/containerd runtime, default for older clusters.  
- **AL2023** — newer, systemd-based, replacing AL2.

**Q21. Why use Bottlerocket?**  
- Smaller attack surface (no package manager, no shell in container version).  
- Immutable root FS.  
- Automatic security updates.  
- Optimized for container workloads.

**Q22. Can you SSH into Bottlerocket nodes?**  
Not by default. Use `admin` container (privileged sidecar) or SSM Session Manager.

**Q23. Launch Template customization with managed node groups:**  
Supported since 2020. Lets you add custom AMI, user-data (extending bootstrap), instance types, storage. Required for advanced use (custom CNI, kernel params, custom image).

**Q24. Node group taints and labels — how applied?**  
Directly via managed node group config (since 2022). Previously had to use `kubelet-extra-args` in bootstrap. Labels for affinity, taints for dedicated workloads (GPU, spot, memory-heavy).

**Q25. Multi-arch node groups (x86 + ARM Graviton):**  
Separate managed node groups per architecture. Helm charts / images must be multi-arch (buildx). Use `nodeAffinity` / taints to schedule correctly.

---

## 5. NETWORKING — VPC CNI DEEP DIVE

**Q26. What is AWS VPC CNI?**  
Default EKS CNI. Assigns each pod a routable VPC IP (from subnet). Uses ENIs attached to node; each ENI contributes a set of IPs. Native AWS networking — pod IPs routable VPC-wide.

**Q27. VPC CNI IP allocation model:**  
- Each node has 1 primary ENI + secondary ENIs.  
- Each ENI has pool of secondary IPs.  
- Total pods/node = (ENIs × IPs/ENI) − 1 (reserved) — subject to max pods per instance type.  
- **Prefix delegation** (2021+) — each ENI gets `/28` prefix (16 IPs) instead of single IPs → 16× more pods per ENI.

**Q28. Why enable prefix delegation?**  
Without it, `m5.large` = 3 ENIs × 10 IPs = 29 pods max. With prefix delegation = 3 × 16 × 9 = hundreds of pods. Essential for high pod density on smaller instances.

**Q29. Custom networking (separate pod CIDR):**  
Pods in different subnet(s) than nodes — typically secondary CIDR (`100.64.0.0/10` CGNAT range). Useful when primary VPC CIDR is IP-limited.

**Q30. VPC CNI WARM_IP_TARGET / MINIMUM_IP_TARGET / WARM_PREFIX_TARGET:**  
Tuning how many IPs/prefixes CNI pre-allocates per node. Trade-off: fast pod startup vs wasted IPs. Common: `WARM_PREFIX_TARGET=1` balances well.

**Q31. SNAT in VPC CNI:**  
By default, pod egress is SNATed to node IP if target is outside VPC. Disable via `AWS_VPC_K8S_CNI_EXTERNALSNAT=true` when using NAT Gateway (preserves pod IP visibility).

**Q32. What is the ENI trunking feature?**  
For ECS `awsvpc` mode on Nitro — trunk ENI carries many branch ENIs. Increases task density. EKS doesn't use directly, but similar efficiency via prefix delegation.

**Q33. IPv6 in EKS:**  
Supported (2021+). Pods get IPv6 from subnet. AWS API endpoints reachable via egress-only IGW + NAT64/DNS64. Solves IP exhaustion at massive scale.

**Q34. Can you mix IPv4 and IPv6 clusters?**  
A cluster is either IPv4 or IPv6 — set at creation, immutable. No dual-stack pods.

---

## 6. ALTERNATIVE CNIs

**Q35. Why choose Calico, Cilium, or other CNIs?**  
- VPC CNI: VPC IP per pod (native AWS), consumes IPs heavily, basic policy.  
- **Calico** — NetworkPolicy + global policies, overlay (VXLAN) saves VPC IPs.  
- **Cilium** — eBPF-based, advanced L7 policy, observability (Hubble), service mesh capabilities, replaces kube-proxy.  
- **Weave** — mostly replaced.

**Q36. Can you run Cilium on EKS?**  
Yes — two modes: chaining (on top of VPC CNI) or replacement (full Cilium). Replacement gives best features but more complex setup.

**Q37. kube-proxy on EKS:**  
Default add-on, uses iptables/IPVS. Cilium can replace it (`kubeProxyReplacement: true`) for faster service routing via eBPF.

---

## 7. POD NETWORKING & SECURITY GROUPS FOR PODS

**Q38. Security Groups for Pods — what is it?**  
Feature allowing you to attach AWS Security Groups to pods (not just nodes). Pod gets its own ENI (branch). Useful for RDS access, compliance.

**Q39. Prerequisites:**  
- Nitro-based instance.  
- VPC CNI `ENABLE_POD_ENI=true`.  
- `SecurityGroupPolicy` CRD defining SG → pod selector.  
- Instance types supporting trunking.

**Q40. Limitations:**  
- Not all instance types.  
- Reduces effective pod density per node (each pod ENI consumes trunking slot).  
- Doesn't work with Fargate.  
- Host-network pods not supported.

**Q41. NetworkPolicy in EKS:**  
VPC CNI (1.14+) supports Kubernetes NetworkPolicy natively. Before, needed Calico. Implements ingress/egress rules between pods using eBPF.

---

## 8. INGRESS CONTROLLERS & AWS LOAD BALANCER CONTROLLER

**Q42. What is AWS Load Balancer Controller?**  
Kubernetes controller that provisions ALB (Ingress) or NLB (LoadBalancer Service) based on annotations. Replaces legacy in-tree `service-type: LoadBalancer` with more features.

**Q43. IngressGroup — what problem does it solve?**  
Multiple Ingress resources sharing a single ALB (cost optimization). Group name in annotation → one ALB with rules merged by priority.

**Q44. TargetType: `ip` vs `instance`:**  
- `instance` (NodePort-based) — uses kube-proxy, double hop.  
- `ip` (pod direct) — ALB/NLB targets pod IPs (via VPC CNI IP). Lower latency, required for Fargate.

**Q45. Required IAM permissions for AWS LB Controller:**  
Large policy including `elasticloadbalancing:*`, `ec2:DescribeSecurityGroups`, `ec2:CreateSecurityGroup`, `iam:CreateServiceLinkedRole`. AWS provides reference JSON. Use IRSA.

**Q46. Common annotations interviewers ask:**  
- `alb.ingress.kubernetes.io/scheme: internet-facing | internal`  
- `alb.ingress.kubernetes.io/target-type: ip | instance`  
- `alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80},{"HTTPS": 443}]'`  
- `alb.ingress.kubernetes.io/certificate-arn: arn:...`  
- `alb.ingress.kubernetes.io/ssl-redirect: '443'`  
- `alb.ingress.kubernetes.io/group.name: my-group`  
- `alb.ingress.kubernetes.io/healthcheck-path: /healthz`  
- `alb.ingress.kubernetes.io/tags: env=prod,team=platform`

**Q47. NLB vs ALB in EKS — when to use:**  
- **ALB** — HTTP(S) apps, path-based routing, WebSockets, WAF integration.  
- **NLB** — TCP/UDP, static IP, extreme performance, preserves client IP at L4.

**Q48. Nginx Ingress vs AWS LB Controller:**  
- **Nginx** — pod-based (runs as Deployment), L7 features, cheaper (one LB), customizable, common in on-prem/K8s-portable setups.  
- **AWS LB Controller** — AWS-native, per-Ingress ALB (or grouped), managed scaling, no pod overhead.  
Your team's choice often based on portability vs AWS-native preference.

---

## 9. SERVICE TYPES IN EKS

**Q49. Kubernetes Service types — EKS behavior:**  
- **ClusterIP** — internal, no LB provisioned.  
- **NodePort** — opens port on each node; rarely used directly in EKS.  
- **LoadBalancer** — provisions CLB (legacy) or NLB via AWS LB Controller.  
- **ExternalName** — CNAME to external DNS.  
- **Headless** — no ClusterIP, direct pod DNS (StatefulSet).

**Q50. Internal vs External LoadBalancer annotation:**  
```yaml
service.beta.kubernetes.io/aws-load-balancer-internal: "true"
service.beta.kubernetes.io/aws-load-balancer-scheme: "internal"
service.beta.kubernetes.io/aws-load-balancer-type: "external"   # use LB Controller NLB
service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
```

**Q51. CoreDNS on EKS:**  
Cluster DNS add-on. Resolves `*.svc.cluster.local`. Configure forwarders for `*.example.com` to Route 53 Resolver. Scale replicas for large clusters.

**Q52. External DNS controller:**  
Reads Ingress/Service annotations, creates Route 53 records automatically. IAM permissions required (`route53:ChangeResourceRecordSets`). Prevents manual DNS drift.

---

## 10. STORAGE IN EKS

**Q53. EBS CSI Driver — what does it do?**  
Provisions EBS volumes for PVCs. Since 2022, EKS managed add-on. Replaces in-tree driver (deprecated in K8s 1.23+).

**Q54. Key EBS CSI features:**  
- Dynamic provisioning  
- Volume snapshots  
- Volume expansion  
- Topology-aware (same AZ as pod)  
- Encryption (KMS)  
- gp3 support (default now)

**Q55. StorageClass example:**  
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
  encrypted: "true"
  kmsKeyId: "arn:aws:kms:..."
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

**Q56. Why `WaitForFirstConsumer`?**  
Delays PV provisioning until pod is scheduled → AZ of pod determined first, then EBS volume created in same AZ. Avoids mismatch where pod can't use volume in different AZ.

**Q57. EFS CSI Driver — use case:**  
Shared filesystem (NFS) across pods in multiple AZs. Good for WordPress, Jenkins shared workspace, ML datasets. Unlike EBS, EFS is multi-AZ natively.

**Q58. EFS Access Points in EKS:**  
Enforce POSIX UID/GID per app. Useful for multi-tenant. Referenced in PV via `volumeHandle: fs-xxx::fsap-yyy`.

**Q59. FSx for Lustre CSI — when to use?**  
HPC / ML training — extreme throughput (100s GB/s). Integrates with S3 (lazy load, auto-export). Pricey but essential for AI workloads. **Your NetApp background is relevant — FSx for NetApp ONTAP also supported via CSI.**

**Q60. Common storage gotcha: Pod stuck in `ContainerCreating` with "failed to attach volume".**  
Typically:  
- PVC pending because no matching PV / StorageClass wrong.  
- EBS volume in different AZ than pod.  
- IAM role missing EBS permissions.  
- Volume still attached to old node (post-failure); force detach.

**Q61. Volume Snapshots:**  
EBS CSI supports `VolumeSnapshot` / `VolumeSnapshotClass`. Used for backup/restore, velero-like workflows.

---

## 11. IRSA (IAM ROLES FOR SERVICE ACCOUNTS)

**Q62. Why IRSA instead of node IAM role?**  
Node role grants all pods on that node same AWS permissions. IRSA scopes permissions to specific ServiceAccount → specific pod — least privilege.

**Q63. IRSA prerequisites:**  
1. OIDC provider associated with cluster (`eksctl utils associate-iam-oidc-provider`).  
2. IAM role with trust policy for that OIDC + specific `sub` claim.  
3. K8s ServiceAccount annotated `eks.amazonaws.com/role-arn: arn:aws:iam::...:role/...`.  
4. Pod uses that ServiceAccount.

**Q64. How does IRSA inject credentials into the pod?**  
EKS Pod Identity Webhook mutates pod spec to:  
- Add env vars `AWS_ROLE_ARN`, `AWS_WEB_IDENTITY_TOKEN_FILE`.  
- Mount projected service account token.  
- AWS SDK calls `AssumeRoleWithWebIdentity` automatically.

**Q65. IRSA trust policy structure:**  
```json
{
  "Effect": "Allow",
  "Principal": { "Federated": "arn:aws:iam::ACCT:oidc-provider/oidc.eks.<region>.amazonaws.com/id/<id>" },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "<oidc-url>:aud": "sts.amazonaws.com",
      "<oidc-url>:sub": "system:serviceaccount:<ns>:<sa>"
    }
  }
}
```

**Q66. SDK version requirement for IRSA:**  
AWS SDK must support `AssumeRoleWithWebIdentity`. Most modern SDKs (2019+) support. Old SDKs fall back to IMDS → get node role → wrong perms.

**Q67. Common IRSA pitfall — IMDS leak:**  
If pod network can reach IMDS (`169.254.169.254`), SDK may prefer it over IRSA. Fix: set `HTTP_TOKENS=required`, `HTTP_PUT_RESPONSE_HOP_LIMIT=1` on nodes so pods (2 hops) can't reach IMDS.

**Q68. Multiple IRSA roles for one workload:**  
Typically one role per ServiceAccount. For multi-role pod, use `AssumeRole` chaining or separate containers with different SA (rare).

---

## 12. EKS POD IDENTITY

**Q69. What is EKS Pod Identity (2023+)?**  
Simpler alternative to IRSA. EKS add-on (`eks-pod-identity-agent`) runs as DaemonSet. Associate IAM role with ServiceAccount via EKS API (not trust policy). No OIDC per cluster.

**Q70. Pod Identity vs IRSA — comparison:**  
| Aspect | IRSA | Pod Identity |
|---|---|---|
| Setup | OIDC provider per cluster | DaemonSet add-on |
| IAM Role Trust | Per cluster OIDC | Single trust for `pods.eks.amazonaws.com` |
| Reuse across clusters | No (different OIDC) | Yes |
| SDK version | Needs recent SDK | Works with older SDKs |
| Region awareness | Manual | Native |
| Session tags | Manual | Built-in (cluster, namespace, SA) |

**Q71. When to still use IRSA?**  
- Clusters created before Pod Identity launch.  
- Cross-cluster role sharing where you've already invested in OIDC pattern.  
- Fine-grained session policies.

**Q72. Pod Identity association command:**  
```bash
aws eks create-pod-identity-association \
  --cluster-name my-cluster \
  --role-arn arn:aws:iam::ACCT:role/MyRole \
  --namespace default \
  --service-account my-sa
```

---

## 13. AUTHENTICATION & AUTHORIZATION

**Q73. How does kubectl authenticate to EKS?**  
1. IAM Authenticator exchanges IAM credentials for a token (`aws-iam-authenticator` or `aws eks get-token`).  
2. Token sent to API server in bearer header.  
3. EKS validates IAM identity via STS.  
4. Maps to Kubernetes user/group per `aws-auth` ConfigMap or Access Entries.

**Q74. Legacy `aws-auth` ConfigMap — what does it do?**  
Maps IAM users/roles to Kubernetes RBAC users/groups. Single ConfigMap in `kube-system`. Critical — delete it = lock yourself out of cluster.

**Q75. New EKS Access Entries / Access Policies (2023+):**  
Replace aws-auth ConfigMap. Manage access via EKS API. Predefined policies (`AmazonEKSClusterAdminPolicy`, `AmazonEKSAdminPolicy`, `AmazonEKSViewPolicy`, `AmazonEKSEditPolicy`). Scoped to namespaces.

**Q76. Why is Access Entries better than aws-auth?**  
- Managed via AWS API (auditable via CloudTrail).  
- No YAML manipulation risk.  
- Programmable via Terraform natively.  
- AWS-defined policies reduce custom RBAC drift.

**Q77. Cluster creator privileges — what's the gotcha?**  
IAM identity that creates cluster becomes admin automatically. If created by CI/CD role, humans can't get in until they bootstrap access. Best practice: use a break-glass role for creation + immediately add team roles.

**Q78. RBAC vs IAM for EKS:**  
- **IAM** — authenticates (who are you?).  
- **RBAC** — authorizes (what can you do?).  
Both required. IAM can authenticate you but RBAC decides permissions.

**Q79. How to grant kubectl access to GitHub Actions?**  
OIDC provider for GitHub → IAM role trusts GitHub OIDC → Access Entry maps that role to EKS cluster admin/edit. No long-lived credentials.

---

## 14. EKS ADD-ONS

**Q80. What is an EKS Add-on?**  
AWS-managed Kubernetes software (kube-proxy, CoreDNS, VPC CNI, EBS CSI, EFS CSI, Pod Identity Agent, ADOT, Snapshot Controller, etc.). AWS handles upgrades, compatibility, configuration.

**Q81. Advantages of add-ons vs self-installed:**  
- Automatic version compatibility check with cluster.  
- One-command updates.  
- AWS support included.  
- Configuration via EKS API.

**Q82. Can you override add-on configuration?**  
Yes via `configurationValues` (JSON/YAML) — similar to Helm values. Examples: VPC CNI with prefix delegation enabled.

**Q83. Add-on vs Helm — migration concerns:**  
Migrating from self-managed (Helm) to add-on requires cleanup of old resources. AWS has migration guides per add-on.

**Q84. Popular community add-ons via Marketplace:**  
Datadog, New Relic, HashiCorp Vault, Tetrate, Kasten K10. Available via EKS Console → Add-ons → Marketplace.

---

## 15. CLUSTER UPGRADES

**Q85. EKS upgrade — the 3 steps:**  
1. **Control plane** — `aws eks update-cluster-version --name <> --kubernetes-version 1.30`.  
2. **Core add-ons** — update VPC CNI, CoreDNS, kube-proxy, EBS CSI to compatible versions.  
3. **Data plane** — update managed node groups / self-managed nodes / Fargate profiles.

**Q86. Upgrade one minor version at a time — why?**  
Kubernetes supports N-1 skew between API server and kubelet. Skipping versions breaks this. Also API deprecations — need to fix incompatible APIs before each step.

**Q87. Common API deprecation pitfalls:**  
- `v1beta1` Ingress → `v1` (1.22).  
- `v1beta1` PodDisruptionBudget → `v1` (1.25).  
- `policy/v1beta1` → `policy/v1`.  
- PodSecurityPolicy removed (1.25) → PSA (Pod Security Admission).

**Q88. Pluto / kube-no-trouble — what do they do?**  
Scan manifests / live cluster for deprecated APIs before upgrade. Run in CI to block PRs using deprecated APIs.

**Q89. Node group upgrade — rolling strategy:**  
Managed node group: max-unavailable (%/absolute), max-unavailable-percentage. AWS drains one node, launches new with new AMI. Respects PodDisruptionBudget.

**Q90. Blue-green cluster upgrade — when to use?**  
Major version jumps, significant CRD changes, extra cautious environments. Create new cluster, migrate workloads gradually, switch DNS, decommission old. Veleros/ArgoCD to sync.

**Q91. Post-upgrade verification checklist:**  
- `kubectl get nodes` — all Ready, new version.  
- Workload health.  
- Ingress / Service LBs working.  
- Metrics and logging intact.  
- HPA/Cluster Autoscaler operational.  
- CRDs compatible.

---

## 16. CLUSTER AUTOSCALER

**Q92. What does Cluster Autoscaler do?**  
Watches unschedulable pods → scales node groups up. Watches underutilized nodes → scales down. Works with ASG tags.

**Q93. CA vs HPA:**  
- **HPA** scales pods (replicas).  
- **CA** scales nodes (capacity).  
Both work together.

**Q94. CA configuration essentials:**  
- IAM role (IRSA) with `autoscaling:DescribeAutoScalingGroups`, `autoscaling:DescribeAutoScalingInstances`, etc.  
- ASG tags: `k8s.io/cluster-autoscaler/<cluster-name>: owned`, `k8s.io/cluster-autoscaler/enabled: true`.  
- Version compatible with K8s version (CA v1.27.x for K8s 1.27).  
- `--balance-similar-node-groups` to spread across AZs.  
- `--skip-nodes-with-local-storage=false` to evict nodes with emptyDir.

**Q95. Why does CA take time to scale up?**  
Pod unschedulable detected → CA decides → ASG launches EC2 → EC2 boot → kubelet registers → pod scheduled. Usually 2–5 min. Use pod priority / over-provisioning to buffer.

**Q96. Over-provisioning pattern:**  
Deploy low-priority "pause" pods with resource requests → forces extra nodes ready. Real workloads preempt pause pods instantly → no CA wait.

---

## 17. KARPENTER

**Q97. What is Karpenter and how is it different from Cluster Autoscaler?**  
Karpenter is a just-in-time node provisioner. Doesn't use ASGs — talks directly to EC2 API to launch correct-sized instances. Launches mixed instance types automatically based on pod requirements.

**Q98. Karpenter vs CA — key benefits:**  
- Faster (60s vs 3–5 min).  
- Bin-packs better (chooses efficient instance type per pod batch).  
- No ASG management.  
- Handles Spot interruption gracefully.  
- Consolidates nodes (replaces with cheaper alternatives).

**Q99. Karpenter concepts:**  
- **NodePool** (earlier `Provisioner`) — constraints: instance types, zones, capacity type (Spot/On-Demand), limits.  
- **EC2NodeClass** (earlier `AWSNodeTemplate`) — AMI family, subnet selector, SG selector, user-data, block device mappings, instance profile.  
- **Consolidation** — proactively replace underutilized nodes.  
- **Drift** — detect node spec differences vs template, replace.

**Q100. Karpenter and Spot — best practices:**  
- Use `karpenter.sh/capacity-type: spot` preference.  
- Diversify instance types (20+) — reduces interruption.  
- Fallback to On-Demand via second NodePool.  
- Handle interruption: Karpenter subscribes to SQS for Spot Interruption Notice → cordons and drains 2 min before termination.

**Q101. Karpenter IAM requirements:**  
Karpenter controller role (IRSA) + EC2 node role (for kubelet). Separate IAM role for node (instance profile).

**Q102. Karpenter consolidation modes:**  
- `WhenUnderutilized` — replaces nodes when cheaper alternative exists.  
- `WhenEmpty` — removes empty nodes only (safer).

**Q103. Common Karpenter issue: pods stuck pending.**  
- NodePool constraints exclude suitable instance type.  
- Subnet tag missing (`karpenter.sh/discovery`).  
- Instance type unavailable in AZ (InsufficientCapacity).  
- IAM role missing EC2 permissions.  
- EC2 quotas hit.

---

## 18. HPA / VPA / KEDA

**Q104. HPA fundamentals:**  
Horizontal Pod Autoscaler scales replicas based on metrics (CPU, memory, custom). Requires metrics-server or Prometheus Adapter.

**Q105. HPA target types:**  
- `Utilization` — % of pod's request.  
- `AverageValue` — absolute value.  
- `Value` — raw metric.

**Q106. HPA custom metrics from CloudWatch:**  
Install `k8s-cloudwatch-adapter` or use Prometheus Adapter + CloudWatch exporter. Then HPA can scale on SQS queue depth, ALB request count, custom app metrics.

**Q107. VPA — what and caveats?**  
Vertical Pod Autoscaler recommends/adjusts pod CPU/memory requests. Modes: `Off`, `Initial`, `Recreate`, `Auto`. Don't use with HPA on CPU/memory (conflict). VPA triggers pod restart — disruptive.

**Q108. KEDA — when to use?**  
Event-driven autoscaling (Kafka lag, SQS depth, Redis list length, Azure Service Bus, etc.). 50+ scalers. Scales to zero (unlike HPA). Commonly used for async workloads.

**Q109. KEDA + SQS example:**  
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
spec:
  scaleTargetRef: { name: worker }
  minReplicaCount: 0
  maxReplicaCount: 100
  triggers:
  - type: aws-sqs-queue
    metadata:
      queueURL: https://sqs...
      queueLength: "5"
      awsRegion: us-east-1
    authenticationRef: { name: keda-aws-creds }
```

---

## 19. OBSERVABILITY

**Q110. Container Insights — what is it?**  
CloudWatch feature for container observability. Collects metrics/logs from cluster via CloudWatch Agent + Fluent Bit. Dashboards: cluster, node, pod, service level.

**Q111. Container Insights cost:**  
Can be expensive at scale (per-metric/log cost). Alternatives: Prometheus + Grafana (cheaper for deep metrics), Datadog, New Relic.

**Q112. ADOT (AWS Distro for OpenTelemetry):**  
AWS-supported OpenTelemetry distribution. Collects traces, metrics. Targets: X-Ray, CloudWatch, Prometheus, Managed Grafana. Replaces legacy xray-daemon.

**Q113. AMP (Amazon Managed Prometheus) + AMG (Amazon Managed Grafana):**  
Serverless Prometheus-compatible metrics store + managed Grafana. Scale to millions of series without managing HA Prometheus. IAM-integrated authentication.

**Q114. Prometheus scraping setup on EKS:**  
- Install kube-prometheus-stack (Helm) or ADOT collector.  
- ServiceMonitor / PodMonitor CRDs for targets.  
- Remote write to AMP or self-hosted Thanos.

**Q115. Tracing with X-Ray:**  
Deploy x-ray daemon as DaemonSet OR use ADOT Collector. App SDK (X-Ray / OpenTelemetry) sends spans. Correlates with CloudWatch logs.

**Q116. FluentBit vs FluentD on EKS:**  
- **FluentBit** — lightweight (C), lower memory, default for Container Insights.  
- **FluentD** — feature-rich (Ruby), more plugins, higher resource usage.  
Most use FluentBit unless legacy.

**Q117. Destination options for logs:**  
CloudWatch Logs, S3 (via Firehose), Kinesis, OpenSearch, Splunk, Datadog. Typical pattern: FluentBit → CW Logs or Firehose → S3 + OpenSearch.

---

## 20. LOGGING DEEP DIVE

**Q118. Control plane logging — which logs?**  
- **api** — API server logs.  
- **audit** — all API calls (compliance-critical).  
- **authenticator** — IAM auth decisions.  
- **controllerManager** — controllers.  
- **scheduler** — pod scheduling decisions.  
Published to CloudWatch Logs `/aws/eks/<cluster>/cluster`.

**Q119. Why enable audit logs?**  
Forensics (who deleted that deployment?), compliance (SOC2, HIPAA), security (GuardDuty uses it for detections).

**Q120. Application pod logs — collection:**  
stdout/stderr → kubelet → node disk (`/var/log/pods`) → FluentBit tails → CW Logs / other destination.

**Q121. Log rotation on nodes:**  
kubelet config `containerLogMaxSize` / `containerLogMaxFiles`. Default 10 MB × 5 files. Important for long-running pods to prevent disk fill.

**Q122. Structured logging best practice:**  
Apps emit JSON logs → FluentBit parses → CW Logs Insights queries. Makes queries efficient vs regex over unstructured text.

---

## 21. SERVICE MESH

**Q123. AWS App Mesh — what is it?**  
AWS-managed service mesh based on Envoy. Sidecar proxy pattern for traffic management, observability, security. Announced deprecation (2024) in favor of other options (Istio on EKS, VPC Lattice).

**Q124. Istio on EKS — install patterns:**  
- istioctl (easiest for dev).  
- Helm chart (production, versioned).  
- Istio Operator (declarative).  
- AWS distro / Tetrate (commercial support).

**Q125. Why use service mesh?**  
- mTLS between services.  
- Traffic splitting (canary, A/B).  
- Circuit breaking, retries, timeouts.  
- Observability (distributed tracing).  
- RBAC at service level.

**Q126. Alternatives:**  
- **Linkerd** — simpler, lightweight.  
- **Cilium Service Mesh** — eBPF-based, no sidecar.  
- **VPC Lattice** — AWS service-level (not pod-level).

---

## 22. SECRETS MANAGEMENT

**Q127. Native Kubernetes Secrets — limitations:**  
Stored in etcd base64 encoded (not encrypted by default — enable KMS envelope). Any pod with RBAC can read them. No rotation.

**Q128. External Secrets Operator (ESO):**  
Syncs secrets from external stores (Secrets Manager, SSM, Vault) to Kubernetes Secrets. CRD `ExternalSecret` + `SecretStore`. Automatic refresh.

**Q129. Secrets Store CSI Driver:**  
Mounts secrets directly as volume (not K8s Secret). AWS provider reads from Secrets Manager / SSM. Closer to "pull on demand." Works with Fargate.

**Q130. ESO vs CSI Driver — pick which?**  
- **ESO** — syncs to K8s Secrets; multiple pods consume; simpler app code.  
- **CSI Driver** — mounts as file; no K8s Secret stored; better for ephemeral/compliance.

**Q131. Secrets rotation pattern:**  
Secrets Manager rotates CMK → ESO/CSI detects → app picks up new secret. App should reload on file change or SIGHUP / restart. Rolling restart via `kubectl rollout restart deployment`.

---

## 23. EKS FARGATE DEEP DIVE

**Q132. What is Fargate on EKS?**  
Serverless data plane. Each pod gets its own microVM (Firecracker). No node management. Pay per pod-second.

**Q133. Fargate profile — what does it do?**  
Defines which pods run on Fargate via namespace + label selectors. Pod matching profile → Fargate; else → regular nodes.

**Q134. Fargate limitations:**  
- No DaemonSets (one pod = one microVM).  
- No privileged mode.  
- No HostPath / HostNetwork.  
- No GPU.  
- Limited instance sizing (0.25–16 vCPU, 0.5–120 GB).  
- Limited storage (20 GB ephemeral + EFS).  
- Slower startup than EC2 nodes.  
- No custom kernel params.  
- Only ALB/NLB target-type `ip`.

**Q135. Fargate pricing vs EC2:**  
Fargate is more expensive per vCPU/GB at high utilization but breakeven for sporadic/bursty workloads. Rule of thumb: <50% utilization → Fargate cheaper; >70% → EC2 cheaper.

**Q136. Fargate logging:**  
Built-in Fluent Bit integration. Configure via ConfigMap in `aws-observability` namespace. Sends to CW Logs, Kinesis Firehose, OpenSearch.

---

## 24. EKS ON OUTPOSTS / ANYWHERE / HYBRID

**Q137. EKS on Outposts:**  
EKS control plane in region + data plane on AWS Outposts rack on-prem. Low-latency to local resources. Or fully local (control + data on Outposts for disconnected ops).

**Q138. EKS Anywhere (EKS-A):**  
Install EKS-compatible Kubernetes on your infra (bare metal, vSphere, Nutanix, Snow). Not managed by AWS; you operate it. Enterprise Subscription available.

**Q139. EKS Hybrid Nodes (2024+):**  
On-prem servers join EKS cluster as nodes (control plane still in AWS). Bridges cloud + on-prem without separate tooling.

**Q140. Use cases for hybrid:**  
- Data residency (regulations).  
- Legacy systems close to app.  
- Edge / disconnected.  
- Burst to cloud during peak.

---

## 25. COST OPTIMIZATION

**Q141. Top 10 EKS cost levers:**  
1. Karpenter / right-sizing instances.  
2. Spot instances with mixed NodePools.  
3. Graviton (ARM64) for 40% savings.  
4. Consolidation (Karpenter).  
5. Fargate for tiny workloads; EC2 for dense.  
6. Savings Plans / Reserved Instances for stable baseline.  
7. Cluster autoscaler scale-down aggressiveness.  
8. gp3 over gp2 for EBS.  
9. CloudWatch log retention / exclude noisy containers.  
10. Delete unused CRDs, namespaces, orphaned PVs.

**Q142. Identify biggest EKS cost drivers:**  
- EC2/Fargate compute (typically 70–80%).  
- EBS volumes (often wasteful — orphaned PVs).  
- NAT Gateway data transfer (ECR pulls, S3).  
- Load Balancers (many Ingress → many ALBs → use IngressGroup).  
- CloudWatch logs ingestion (Container Insights).

**Q143. Kubecost / OpenCost on EKS:**  
Provides per-namespace / per-pod / per-deployment cost visibility. Reads Prometheus + AWS pricing. Helps chargeback and optimization.

**Q144. How to reduce NAT Gateway costs:**  
- VPC Endpoint for ECR (interface), ECR public, S3 (gateway).  
- Pull-through cache (ECR) for public images.  
- In-region ECR replication.  
- Private endpoint for CloudWatch Logs.

---

## 26. SECURITY & COMPLIANCE

**Q145. Pod Security Admission (PSA) — what replaces PSP?**  
PSP removed in 1.25. PSA is built-in controller with 3 profiles: `privileged`, `baseline`, `restricted`. Enforced via namespace labels.

**Q146. Admission controllers common in EKS:**  
- **PSA** (native)  
- **OPA/Gatekeeper** — policy as code  
- **Kyverno** — Kubernetes-native policy engine  
- **AWS LB Controller webhook**  
- **ValidatingAdmissionPolicy (CEL, 1.30+)** — native replacement for simple policies

**Q147. Image scanning options:**  
- **ECR scan on push** (Amazon Inspector integration, free basic).  
- **Trivy / Grype** — run in CI.  
- **Aqua / Prisma / Snyk** — commercial.

**Q148. Runtime security:**  
- **Amazon GuardDuty EKS Protection** — detects anomalous API calls, malware.  
- **Falco** — eBPF runtime detection.  
- **Calico Enterprise** — network + runtime.  
- **Sysdig Secure** — commercial.

**Q149. CIS EKS benchmark:**  
Security baseline published by CIS. Tools like `kube-bench` verify. Cover control plane config, worker node config, policies, RBAC.

**Q150. Compliance considerations (HIPAA, PCI, FedRAMP):**  
- Envelope encryption for secrets (KMS CMK).  
- Audit logs to S3 (immutable).  
- Network policies / SG for pods.  
- Private API endpoint only.  
- Pod security policies.  
- IRSA least privilege.  
- Control Tower + Config rules.

**Q151. Secrets in environment variables vs volumes:**  
Env vars visible in `kubectl describe pod` (to anyone with describe perms). Volumes less exposed. Prefer CSI Driver mounting to file.

---

## 27. UPGRADES & DISASTER RECOVERY

**Q152. Velero for EKS backup/restore:**  
- Backup Kubernetes resources + PV snapshots.  
- Stores in S3 + EBS/EFS snapshots.  
- Restore to same or different cluster/region.  
- Used for DR, migration, cluster upgrades.

**Q153. EKS cluster DR strategies:**  
- **Pilot light** — minimal cluster in DR region; scale up on failover.  
- **Warm standby** — fully running but smaller scale.  
- **Multi-site active-active** — two clusters both live (DNS/Global Accelerator).  
- **Backup-restore** — Velero restores on demand (highest RTO).

**Q154. PV DR patterns:**  
- Velero + CSI snapshots (EBS cross-region).  
- EFS replication (cross-region).  
- Application-level replication (DB replicas).

**Q155. Control plane failure — what happens to workloads?**  
Existing pods keep running (kubelet has manifests). New deployments/scaling blocked until control plane recovers. AWS auto-heals within minutes.

---

## 28. GITOPS ON EKS

**Q156. ArgoCD on EKS — common setup:**  
- Install via Helm / ArgoCD Operator.  
- Public or private ingress (typically private + OIDC SSO).  
- Configure Git repo credentials.  
- App of Apps pattern for multiple apps.  
- Sync waves / hooks for ordering.

**Q157. Flux v2 on EKS:**  
Alternative to ArgoCD. CRDs: `GitRepository`, `Kustomization`, `HelmRelease`. Simpler, CLI-driven. Works well with small teams.

**Q158. GitOps promotion patterns:**  
- Branch per env (dev/staging/prod).  
- Folder per env.  
- Kustomize overlays.  
- Image update automation (ArgoCD Image Updater / Flux ImageRepository).

**Q159. Secrets in GitOps — how to handle?**  
- **SOPS** (encrypted secrets in Git).  
- **Sealed Secrets** (Bitnami — encrypted with cluster key).  
- **External Secrets** (reference only in Git, value in Secrets Manager).

**Q160. Multi-cluster ArgoCD:**  
Hub ArgoCD manages many clusters. Register target clusters with service accounts. Useful for fleet management (100+ clusters).

---

## 29. CI/CD INTEGRATION

**Q161. EKS with Jenkins:**  
- Jenkins agents as Kubernetes pods (Kubernetes plugin).  
- Dynamic scaling per build.  
- IRSA for agent pod accessing ECR, etc.  
- Jenkins controller itself in EKS (StatefulSet + EFS PV).

**Q162. EKS with GitHub Actions:**  
- Self-hosted runners in EKS via Actions Runner Controller (ARC).  
- Ephemeral runners per job.  
- IRSA for runner pods.  
- OIDC federation → AWS roles.

**Q163. GitOps vs Push-based CD:**  
- Push (Jenkins/ArgoCD external `kubectl apply`) — CI has cluster access.  
- Pull/GitOps — cluster pulls from Git; no cluster credentials outside.  
GitOps preferred for security + audit.

**Q164. Helm vs Kustomize — when to use which?**  
- **Helm** — package manager, templating, versioning, release history. Good for third-party (nginx, cert-manager, prometheus).  
- **Kustomize** — overlay-based, no templating, native kubectl. Good for internal apps across envs.  
- Often combined: Helm render → Kustomize patch.

---

## 30. SCENARIO-BASED QUESTIONS

**Q165. Design: EKS platform for 50 teams, multi-tenant.**  
- Shared cluster with namespaces per team.  
- ResourceQuotas + LimitRanges per namespace.  
- NetworkPolicies isolating namespaces.  
- RBAC per team (Access Entries).  
- OPA/Kyverno policies (enforce labels, block `:latest` tag, PSA restricted).  
- Kubecost for chargeback.  
- Observability stack shared (AMP + AMG).  
- ArgoCD app-of-apps per team.  
- Karpenter NodePools per team (or shared with limits).

**Q166. Migrate 200 microservices from on-prem K8s to EKS.**  
- Audit manifests — API deprecations, custom resources.  
- Fix storage (NFS → EFS CSI, iSCSI → EBS CSI).  
- Fix networking (overlay → VPC CNI or keep Calico).  
- Auth (LDAP → IAM + Access Entries).  
- Ingress migration (nginx → AWS LB Controller).  
- Image registry (Harbor → ECR).  
- CI/CD integration.  
- Runbook: move non-stateful first, stateful with Velero + app-level replication.

**Q167. Dev team's pods getting OOMKilled randomly. How to debug?**  
1. `kubectl describe pod` — look for `OOMKilled` reason.  
2. Check requests/limits — likely limit too low OR request underestimated.  
3. Container Insights / Prometheus for memory trend.  
4. App memory leak? heap dump (for JVM).  
5. VPA recommendations.  
6. Adjust resources + redeploy.

**Q168. Node keeps going to `NotReady` every few hours.**  
- Kubelet logs (`journalctl -u kubelet`).  
- Check CNI daemonset health (aws-node pod).  
- IMDSv2 hop-limit breaking kubelet?  
- EBS volume detach (local disk full).  
- Memory pressure eviction.  
- Spot interruption (not reported as NotReady — just terminated).  
- Upgrade to Bottlerocket.

**Q169. How to achieve zero-downtime deploys for a stateful app on EKS?**  
- StatefulSet with `updateStrategy: RollingUpdate`.  
- PodDisruptionBudget `maxUnavailable: 1`.  
- Readiness probes strict.  
- Preload data / warm cache.  
- Service mesh (Istio) for traffic shift.  
- Persistent storage across restart (EBS/EFS).

**Q170. Compliance: pod must not access internet directly.**  
- NetworkPolicy denying egress to 0.0.0.0/0.  
- Allow only VPC CIDR + specific endpoints.  
- Egress proxy (Squid) pods with allowlist.  
- Or: private subnet + no NAT route for specific pods (SG for pods).

**Q171. Dev wants 2 minute pod boot → cluster autoscaler takes 5+.**  
- Karpenter (faster launch).  
- Over-provisioning with low-priority pause pods.  
- Pre-warmed node pool.  
- Pre-pulled images on nodes (bottlerocket image cache).  
- Reduce image size.

**Q172. Global app: EKS clusters in us-east-1 and eu-west-1. Route users.**  
- Route 53 latency routing or Global Accelerator.  
- Global Accelerator for TCP/UDP (static IPs).  
- Cross-region service discovery: ExternalDNS populates Route 53.  
- Data: Aurora Global Database or DynamoDB Global Tables.  
- Replicate secrets via Secrets Manager replication.

**Q173. CFO wants 30% cluster cost reduction.**  
- Karpenter consolidation.  
- Graviton migration (if images support ARM).  
- Spot for non-critical (stateless APIs, batch).  
- Audit: orphan PVs, stale LBs, unused namespaces.  
- Kubecost — find over-provisioned workloads (request vs actual).  
- Lifecycle for CloudWatch logs.  
- Savings Plan baseline.

**Q174. New requirement: 100,000 pods in one cluster.**  
- Check control plane quotas (soft limits; increase).  
- Use prefix delegation for IP density.  
- Multiple large node groups.  
- etcd performance considerations (EKS handles).  
- Consider splitting into multiple clusters (better blast radius).  
- Cilium for scaling kube-proxy.

**Q175. Implementing blue/green deployment for microservice.**  
- Argo Rollouts (CRD replacing Deployment).  
- Two ReplicaSets (blue + green); Service switches.  
- Or: separate namespaces + Ingress weighted routing.  
- Traffic analysis (% error, latency) before full switch.

**Q176. Secret leaked via audit logs (pod env var). Remediate.**  
- Rotate secret immediately (Secrets Manager force rotate).  
- Update all consumers via External Secrets refresh.  
- Redact audit log entries (purge CW Logs).  
- Move secret to volume mount going forward.  
- Investigate blast radius.

---

## 31. TROUBLESHOOTING QUESTIONS

**Q177. `kubectl get nodes` shows NotReady — debug checklist:**  
1. `kubectl describe node` — conditions (MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable).  
2. SSM / SSH to node → `journalctl -u kubelet -f`.  
3. Check `aws-node` pod running (VPC CNI).  
4. `kubelet` process alive.  
5. `/var/lib/kubelet/` disk free.  
6. IMDS accessible from kubelet.  
7. Control plane SG allows node.

**Q178. Pod stuck in `Pending`:**  
1. `kubectl describe pod` — Events section.  
2. "Insufficient cpu/memory" → scale nodes or reduce requests.  
3. "no nodes available" → nodeSelector/tolerations/affinity blocking.  
4. PVC pending → storage class / PV issue.  
5. Image pull backoff earlier → not really pending.  
6. Admission webhook failing (check controller).

**Q179. Pod `CrashLoopBackOff`:**  
1. `kubectl logs <pod> --previous` for last crash.  
2. `kubectl describe` for ExitCode.  
3. Common: liveness probe failing, config missing, wrong image.  
4. `kubectl exec` (if startup enough) to inspect.  
5. ConfigMap/Secret missing → pod fails.

**Q180. `ImagePullBackOff`:**  
- ECR auth: node instance profile / IRSA permissions.  
- Image name typo.  
- Wrong region in ECR URL.  
- Private registry secret missing (imagePullSecrets).  
- Rate limit (Docker Hub).  
- Kernel/arch mismatch (x86 image on ARM node).

**Q181. LoadBalancer service stuck `<pending>`:**  
- AWS LB Controller not running / crashed.  
- IRSA missing perms.  
- Subnet tags missing (`kubernetes.io/role/elb`).  
- Quota exceeded (ALB limit).  
- Security group limit reached.

**Q182. DNS resolution failing inside pods:**  
- CoreDNS pods healthy?  
- `kubectl get svc kube-dns -n kube-system` exists?  
- NetworkPolicy blocking UDP 53.  
- Node DNS config (`/etc/resolv.conf`).  
- `ndots:5` issue causing slow resolution — use FQDN or tune.

**Q183. IRSA working via CLI but app fails:**  
- SDK version too old (web identity support <2019).  
- App using wrong credential provider.  
- Pod hit IMDS (node role wrong perms).  
- IRSA session expired mid-job (default 1 hour; renew with SDK).

**Q184. Karpenter not provisioning nodes:**  
- Controller pod logs.  
- NodePool/EC2NodeClass validation errors.  
- Subnet discovery tags missing.  
- IAM controller role missing perms.  
- `karpenter.sh/do-not-disrupt` annotations blocking.

**Q185. `kubectl` command timing out:**  
- Cluster endpoint access (public allowlist / private VPC).  
- kubeconfig expired STS token — re-run `aws eks update-kubeconfig`.  
- Control plane overload (rare, AWS scales).  
- Local DNS issue.

**Q186. Cluster upgrade fails midway:**  
- Check CloudTrail for `UpdateClusterVersion` event.  
- Add-ons incompatibility (must update CNI/kube-proxy first).  
- Deprecated APIs — workloads fail after upgrade.  
- Rollback: AWS doesn't support; re-apply old manifests if workloads broke.

**Q187. PVC in Pending with "waiting for pod":**  
Normal if StorageClass uses `WaitForFirstConsumer`. PV created only when pod schedules. Pod schedule failing separately? Check events.

**Q188. High API server latency:**  
- Too many CRDs / watches (slim down).  
- Rogue controller opening many watches.  
- Large ConfigMap/Secret updates cascading.  
- Scale control plane: AWS auto-scales; contact support if persists.

---

## 32. REAL-TIME PRODUCTION ISSUES

**Q189. Pods die randomly with exit code 137 during traffic spikes.**  
137 = SIGKILL (OOMKilled). Memory limit too aggressive. Check:  
- `kubectl top pods`.  
- Container Insights memory graph.  
- Heap/leak analysis.  
- Increase limit or optimize app memory.  
- HPA to scale horizontally before OOM.

**Q190. ECR image pull throttled across cluster.**  
- ECR has API rate limits.  
- Fix: image pull credentials cache; stagger deployments; use image pull secrets w/ secrets rotation; ECR pull-through cache.

**Q191. aws-node pod CrashLooping after CNI update.**  
- Version incompatible with cluster.  
- IAM perms changed.  
- ENI attach limit reached.  
- Rollback CNI version via EKS console.  
- Check aws-node logs: `kubectl logs -n kube-system -l k8s-app=aws-node`.

**Q192. Secret rotation broke deployment (app didn't pick new value).**  
- K8s Secrets mounted as file — updated within 60s.  
- Env vars → only updated on pod restart.  
- Solution: rollout restart on secret change (watch with Reloader operator).

**Q193. ArgoCD OutOfSync repeatedly — manifests keep drifting.**  
- Controller mutating fields (e.g., HPA changing replicas).  
- Use `ignoreDifferences` in ArgoCD app.  
- Ensure sync policy includes these fields as managed or ignored.

**Q194. Pod volumes not mounting after node replaced.**  
- Volume still attached to old (terminating) node.  
- Force detach: `aws ec2 detach-volume --volume-id --force`.  
- Improve: use `pod.spec.priorityClassName` for safer eviction ordering; PV reclaim policy `Retain` for critical data.

**Q195. Node cost suddenly 3× — only using 40% of pods.**  
- Karpenter NodePool misconfigured; launching huge instances.  
- Consolidation disabled.  
- Spot unavailable → On-Demand fallback ate money.  
- Review CloudWatch EC2 instance types launched.

**Q196. Service mesh sidecar leaking memory.**  
- Envoy stats histogram config too broad.  
- Tune `stats_matcher` / `retention` settings.  
- Restart daemonset regularly as short-term band-aid.  
- Upgrade mesh version.

**Q197. Cluster autoscaler can't scale down — nodes empty but held.**  
- PDBs preventing eviction.  
- Pods without controller (naked pods).  
- `cluster-autoscaler.kubernetes.io/safe-to-evict=false` annotation.  
- System pods on that node (DaemonSets only should be OK).

**Q198. CloudWatch agent pods filling `/var/log` on node, causing DiskPressure.**  
- Verbose log spam in app.  
- FluentBit not keeping up → logs buffer.  
- Increase node disk (EBS gp3 expand).  
- Tune log rotation.

**Q199. EKS upgrade left us with deprecated API errors.**  
- Run `kubectl convert` / `kubent` to find resources still using old API.  
- Bulk re-apply with updated API version.  
- Check Helm chart versions support new API.

**Q200. SOC2 auditor wants proof of least privilege per workload.**  
- IRSA role inventory: each SA → role → policy.  
- Access Analyzer unused findings.  
- Automate review: Terraform outputs + policy linting in CI.  
- Kyverno policy preventing pods without ServiceAccount IRSA annotation.

---

## 33. DEVOPS / IaC FOCUSED EKS QUESTIONS

**Q201. Terraform module for EKS — pick community or custom?**  
`terraform-aws-modules/eks/aws` — battle-tested, feature-rich, maintained. Custom only if you need unusual constraints. Tag onto company Terraform module layer.

**Q202. Terraform state for EKS — best practices:**  
- Separate state per cluster / account / env.  
- Remote backend (S3 + DynamoDB lock).  
- State encryption (KMS CMK).  
- Workspace or separate folder per env.  
- CI/CD applies with assumed role (OIDC).

**Q203. Avoid destroying cluster by accident:**  
- `prevent_destroy = true` lifecycle.  
- Separate Terraform workspace for cluster vs apps.  
- PR reviews for infra changes.  
- Drift detection + alerting.  
- SCP to deny `eks:DeleteCluster` for non-admins.

**Q204. Managing EKS add-ons via IaC:**  
- Terraform `aws_eks_addon` resource.  
- Version pinning.  
- `resolve_conflicts_on_update: OVERWRITE` for managed configs.  
- configuration_values in JSON.

**Q205. Managing K8s resources via Terraform:**  
- `kubernetes` provider + `kubectl` provider.  
- Works, but slow + state conflicts.  
- Better: Terraform for infra (VPC, EKS, IAM, add-ons), GitOps (ArgoCD/Flux) for app deployments.

**Q206. Bootstrap: Terraform creates EKS, ArgoCD needs to be installed. Chicken-and-egg.**  
- Terraform installs ArgoCD via Helm provider as final step.  
- ArgoCD manages itself (self-management pattern) going forward.  
- Or: separate bootstrap phase with scripts.

**Q207. GitHub Actions deploying to EKS — OIDC setup:**  
- IAM role trusts GitHub OIDC with `sub` = repo + branch.  
- Access Entry maps role to cluster admin/namespace.  
- Action uses `aws-actions/configure-aws-credentials` → `aws eks update-kubeconfig` → `kubectl apply`.

**Q208. Helm chart versioning in production:**  
- Pin exact version (`version: 1.2.3`, not `~1.2`).  
- OCI registry (ECR) for chart storage.  
- Signed charts (cosign).  
- ArgoCD tracks chart version from Git.

**Q209. Secrets in Terraform for EKS workloads:**  
- Don't pass secrets through Terraform. Use:  
  - Terraform creates Secret in Secrets Manager (value from TF Cloud var).  
  - App references via External Secrets.  
- Terraform state encrypted, but still leak risk.

**Q210. Testing Terraform EKS changes:**  
- `terraform plan` PR comment.  
- `terratest` for integration (spins test cluster).  
- `checkov` / `tfsec` for security scan.  
- Dry-run in dev account.

---

## BONUS: 20 RAPID-FIRE ONE-LINERS

1. Default EKS K8s version? → Latest supported.  
2. Max nodes per EKS cluster? → Soft 1,000; scale beyond with support.  
3. Max pods per node on m5.large w/o prefix delegation? → ~29.  
4. With prefix delegation? → 110 (K8s default cap).  
5. Does EKS include etcd backup automatically? → Yes, AWS-managed.  
6. Can you SSH to EKS control plane? → No.  
7. Minimum AZs for EKS? → 2.  
8. Does EKS support Windows nodes? → Yes (managed node groups).  
9. Kubernetes version skew policy? → N-1 between API server and kubelet.  
10. Default CNI? → AWS VPC CNI.  
11. Default CoreDNS replicas? → 2 (scale for large clusters).  
12. Fargate pod min size? → 0.25 vCPU / 0.5 GB.  
13. Fargate pod max size? → 16 vCPU / 120 GB (2024 limits).  
14. IRSA session duration default? → 1 hour (max 12).  
15. Can one pod assume multiple IAM roles? → Not directly (use chaining).  
16. Managed node group max nodes? → 450 per group (soft).  
17. Spot percentage in mixed ASG? → Configurable 0–100%.  
18. How many subnets recommended? → 6 (3 public + 3 private across AZs).  
19. Can you change VPC CNI to Calico after creation? → Yes, but complex migration.  
20. EKS pod logs retention default? → Kubelet local; depends on log agent config.

---

## COMMON "WHY" FOLLOW-UPS

- Why EKS over self-managed K8s (kops)?  
- Why IRSA over node IAM role?  
- Why Pod Identity over IRSA (new)?  
- Why Karpenter over Cluster Autoscaler?  
- Why Bottlerocket over AL2?  
- Why GitOps over push-based CD?  
- Why Spot instances risky for stateful workloads?  
- Why private API endpoint matters for compliance?  
- Why prefix delegation is game-changing for IP density?  
- Why blue-green cluster upgrade for major version jumps?

---

## SUGGESTED 4-WEEK PREP PLAN (for your ROSA/K8s background)

1. **Week 1** — Fundamentals, control plane, node types, VPC CNI (Q1–Q41).  
2. **Week 2** — AWS LB Controller, storage, IRSA, Pod Identity, Access Entries (Q42–Q84).  
3. **Week 3** — Upgrades, autoscaling, Karpenter, observability, security (Q85–Q155).  
4. **Week 4** — GitOps, CI/CD, scenarios, troubleshooting, production (Q156–Q210).  
5. Hands-on: Terraform EKS → Karpenter → ArgoCD → sample microservice with IRSA + External Secrets + ALB Ingress.  
6. Practice explaining: "How is EKS different from ROSA/OpenShift?" (hugely valuable given your background).  
7. Always close with: **cost, security, availability, observability, automation, multi-tenancy**.

---

*End of Document – 210+ unique questions. No duplication with EC2/VPC/IAM/S3 banks. Good luck!*
