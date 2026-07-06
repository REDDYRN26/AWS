# AWS EC2 – Complete Interview Question Bank
**Target Audience:** 3–4+ years experience DevOps / Cloud / SRE engineers  
**Scope:** All EC2 modules + Real-time + Scenario-based + Troubleshooting  
**Source mix:** MNC interview patterns (TCS, Infosys, Wipro, Cognizant, Accenture, Deloitte, Capgemini, Amazon, Microsoft, IBM, HCL, LTIM, Mphasis) + LinkedIn shared questions + real production issues.

---

## TABLE OF CONTENTS
1. EC2 Fundamentals
2. Instance Types & Families
3. AMI (Amazon Machine Image)
4. EBS (Elastic Block Store)
5. Instance Store / Ephemeral Storage
6. Security Groups & NACL
7. Key Pairs & SSH
8. Elastic IP, ENI & Networking
9. Placement Groups
10. Auto Scaling Group (ASG)
11. Launch Template vs Launch Configuration
12. Load Balancers (ALB / NLB / CLB / GWLB)
13. User Data & Instance Metadata (IMDSv1 vs IMDSv2)
14. IAM Roles for EC2
15. Pricing Models (On-Demand, Reserved, Spot, Savings Plans, Dedicated)
16. CloudWatch Monitoring & Alarms
17. Systems Manager (SSM)
18. EC2 Image Builder
19. Hibernation, Stop, Terminate lifecycle
20. Scenario-Based Questions
21. Troubleshooting Questions
22. Real-time Production Issues
23. DevOps-focused EC2 Questions (CI/CD, Terraform, Ansible)

---

## 1. EC2 FUNDAMENTALS

**Q1. What is EC2 and why is it called "elastic"?**  
Elastic Compute Cloud – virtual servers in AWS. "Elastic" because you can scale compute up/down on demand (vertically via instance resize, horizontally via ASG).

**Q2. What happens internally when you click "Launch Instance"?**  
AWS selects a physical host in the chosen AZ → allocates CPU/RAM via Nitro hypervisor → attaches ENI to VPC subnet → attaches root EBS from AMI snapshot → runs user-data → instance enters `running` state. Status checks (System + Instance) follow.

**Q3. Difference between `stop`, `reboot`, `hibernate`, `terminate`?**  
- **Reboot** – same host, no data loss, no public IP change.  
- **Stop** – instance moves off host, RAM cleared, EBS retained, public IP released (unless EIP).  
- **Hibernate** – RAM dumped to root EBS, restored on start (needs encrypted EBS, supported instance types).  
- **Terminate** – instance destroyed, root EBS deleted by default (unless `DeleteOnTermination=false`).

**Q4. What are EC2 Status Checks? System vs Instance check?**  
- **System status check** – AWS infrastructure (host hardware, network, power). If fails → AWS responsibility, recover by stop/start (moves to new host).  
- **Instance status check** – OS-level (kernel panic, exhausted memory, wrong network config). Customer responsibility.

**Q5. What is Nitro System?**  
AWS's next-gen hypervisor – offloads virtualization (network, storage, security) to dedicated hardware cards → near bare-metal performance, better security isolation.

**Q6. Can you change instance type while running?**  
No. Must stop → change instance type → start. Exception: some live migration for Graviton not standard. ASG rolling update handles this in production.

**Q7. What are the EC2 instance states and transitions?**  
`pending → running → stopping → stopped → shutting-down → terminated`. Also `rebooting`. Billing stops in `stopped` (except root EBS and EIP if not attached).

**Q8. What region/AZ considerations matter for EC2?**  
- Latency to users  
- Data residency/compliance  
- Service availability (some instance types only in certain regions)  
- Cost (regions have different pricing)  
- Resilience (multi-AZ)

---

## 2. INSTANCE TYPES & FAMILIES

**Q9. Explain the EC2 instance family naming convention (e.g., `m5a.2xlarge`).**  
- **m** – family (general purpose)  
- **5** – generation  
- **a** – attribute (a = AMD, g = Graviton/ARM, n = network optimized, d = NVMe instance store, en = enhanced network)  
- **2xlarge** – size

**Q10. Map each family to workload:**  
- **T (t3, t4g)** – burstable, dev/test, small web  
- **M (m5, m6i, m7g)** – general purpose, balanced  
- **C (c5, c6i, c7g)** – compute-intensive, batch, gaming  
- **R (r5, r6i)** – memory-intensive, in-memory DBs  
- **X, z1d** – extreme memory (SAP HANA)  
- **I (i3, i4i)** – NVMe storage, NoSQL/DW  
- **D (d2, d3)** – dense HDD, Hadoop  
- **P, G, Inf, Trn** – GPU/ML  
- **F1** – FPGA  
- **H1** – HDD streaming

**Q11. What is a burstable instance? How do CPU credits work?**  
T-family earns CPU credits when below baseline, spends them during bursts. `standard` mode stops bursting at zero credits; `unlimited` mode charges extra for sustained burst. Monitor `CPUCreditBalance` in CloudWatch.

**Q12. Graviton vs x86 – when to choose Graviton?**  
Graviton (ARM64) = ~40% better price/performance for many workloads. Good for containerized apps, Java, Go, Python, Nginx. Pitfalls: legacy x86 binaries, some commercial licenses, custom drivers.

**Q13. What is a bare-metal instance?**  
Direct access to physical CPU/memory without hypervisor (e.g., `m5.metal`). Use cases: workloads that need hardware features, hypervisor nesting, license compliance.

**Q14. What is EC2 instance sizing vs right-sizing?**  
**Sizing** – initial selection. **Right-sizing** – ongoing process (via Compute Optimizer / CloudWatch) to match actual utilization. Key metric: `CPUUtilization`, `MemoryUtilization` (CWAgent), network, IOPS.

---

## 3. AMI (AMAZON MACHINE IMAGE)

**Q15. What is an AMI and what does it contain?**  
Template containing root volume snapshot + launch permissions + block device mapping. Includes OS, installed apps, configs.

**Q16. Types of AMI based on storage:**  
- **EBS-backed** – root volume is EBS snapshot, can stop/start.  
- **Instance store–backed** – root is ephemeral, cannot stop (only terminate).

**Q17. How do you create a custom (golden) AMI?**  
1. Launch base instance → configure/install → run tests  
2. (Optional) Sysprep/cloud-init cleanup  
3. `aws ec2 create-image --instance-id ... --name ...`  
4. Tag, encrypt, share, copy cross-region  
5. Use in Launch Template

**Q18. How to share AMI across accounts and regions?**  
- Cross-account: modify AMI permissions + share underlying snapshot; for encrypted AMIs, share KMS key too.  
- Cross-region: `aws ec2 copy-image`.  
- For org-wide distribution: **AWS Resource Access Manager (RAM)** or **EC2 Image Builder** with distribution settings.

**Q19. What is EC2 Image Builder?**  
Managed service for building, testing, and distributing AMIs and container images in a pipeline. Components: Image Recipe → Infrastructure Configuration → Distribution Configuration → Pipeline.

**Q20. Can you modify a running AMI? How do you patch golden images?**  
AMI is immutable. Workflow:  
- Launch instance from old AMI → patch → create new AMI version → update Launch Template → trigger ASG rolling update.  
- Automated via Image Builder + SSM Patch Manager.

**Q21. Deprecated vs Deregistered vs Disabled AMI?**  
- **Deprecate** – hides from default listing after a date; still usable.  
- **Disable** – blocks new launches but existing instances keep running.  
- **Deregister** – permanently removes AMI (instances keep running, snapshots remain).

**Q22. What's an AMI block device mapping?**  
Definition of volumes attached at launch – root volume, extra EBS, ephemeral. Controls size, type, encryption, `DeleteOnTermination`.

---

## 4. EBS (ELASTIC BLOCK STORE)

**Q23. Explain all EBS volume types with use cases:**  
- **gp3** – default, baseline 3000 IOPS / 125 MB/s, independently scalable. Cheaper than gp2 for same performance.  
- **gp2** – legacy, IOPS tied to size (3 IOPS/GB).  
- **io2 / io2 Block Express** – mission-critical, up to 256K IOPS, 99.999% durability.  
- **io1** – legacy provisioned IOPS.  
- **st1** – throughput HDD, big-data, logs.  
- **sc1** – cold HDD, infrequent access.  
- **Magnetic (standard)** – previous gen.

**Q24. gp2 vs gp3 – why migrate?**  
gp3 is ~20% cheaper, supports independent IOPS/throughput scaling without resizing volume. No burst credit model.

**Q25. EBS vs Instance Store:**  
| Feature | EBS | Instance Store |
|---|---|---|
| Persistence | Yes | Lost on stop/terminate |
| Attach/Detach | Yes | No |
| Snapshots | Yes | No |
| AZ-bound | Yes (attach in same AZ) | Local to host |
| Performance | Network-attached | Physically attached (faster) |

**Q26. Can an EBS volume be attached to multiple instances?**  
Normally no (AZ-bound, single attach). Exception: **io1 / io2 Multi-Attach** – up to 16 Nitro instances in same AZ, requires cluster-aware FS (not ext4/xfs).

**Q27. How do EBS snapshots work internally?**  
Incremental, block-level copies stored in S3 (AWS-managed). First snapshot = full, subsequent = changed blocks only. Deleting an intermediate snapshot doesn't break chain (AWS relinks).

**Q28. How do you automate EBS snapshots?**  
- **Data Lifecycle Manager (DLM)** – policy-based.  
- **AWS Backup** – centralized across services.  
- **Lambda + EventBridge** – custom logic.  
- Cross-region/account copy for DR.

**Q29. How do you resize an EBS volume without downtime?**  
1. `aws ec2 modify-volume --size N`  
2. Wait for `optimizing` → `completed`  
3. Grow partition: `growpart /dev/nvme0n1 1`  
4. Grow FS: `resize2fs` (ext4) or `xfs_growfs` (xfs)  
No reboot needed on Nitro.

**Q30. Can you shrink an EBS volume?**  
Directly, no. Workaround: create smaller volume → rsync/dd data → swap.

**Q31. EBS encryption – key points:**  
- Uses KMS (AWS-managed `aws/ebs` or CMK)  
- Encrypts data at rest, in transit between instance and volume, snapshots, AMIs  
- Cannot change encryption on existing volume; must create encrypted snapshot/copy  
- Account default: "Always encrypt new EBS volumes" setting

**Q32. How do you move an EBS volume across AZs / regions?**  
Snapshot → copy snapshot to target region/AZ → create volume from snapshot → attach.

**Q33. What is EBS Fast Snapshot Restore (FSR)?**  
Eliminates the "lazy-load" latency when creating volumes from snapshots – volumes deliver full performance immediately. Costs per AZ.

**Q34. EBS IOPS vs Throughput – how to decide?**  
- **IOPS** – small random I/O (databases, transactional).  
- **Throughput** – large sequential I/O (logs, video).  
- Choose volume type accordingly (io2 for IOPS, st1 for throughput).

**Q35. What are EBS burst balance and when do you care?**  
gp2 and st1/sc1 use burst credits. `BurstBalance` CloudWatch metric – if hits 0, performance drops to baseline. Migrate to gp3 to eliminate burst dependency.

---

## 5. INSTANCE STORE / EPHEMERAL STORAGE

**Q36. When would you use instance store?**  
Scratch data, caches, buffers, distributed replicated workloads (Cassandra, HDFS, Elasticsearch) where data loss on stop is acceptable.

**Q37. Why is instance store not visible after stop/start?**  
Physical disks on host – moving host = losing disks. Only reboot preserves them.

**Q38. How do you mount instance store on Linux (NVMe)?**  
```bash
lsblk                              # identify /dev/nvme1n1
mkfs.xfs /dev/nvme1n1
mkdir /mnt/scratch
mount /dev/nvme1n1 /mnt/scratch
```
Automate via user-data; do NOT add to `/etc/fstab` without `nofail` (instance fails to boot if ephemeral missing).

---

## 6. SECURITY GROUPS & NACL

**Q39. SG vs NACL – 10 differences interviewers love:**  
| Aspect | Security Group | NACL |
|---|---|---|
| Level | Instance (ENI) | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Order | All evaluated | Numbered, first match |
| Default | Deny all inbound | Allow all (default NACL) |
| Apply on | Create/modify any time | Any time |
| Scope | Can reference SG IDs | Only CIDR |
| Limit | 5 SGs/ENI, 60 rules | 20 rules default |
| Return traffic | Auto-allowed | Must allow explicitly |
| Use case | App-level filtering | Subnet-level coarse filter |

**Q40. What does "stateful" mean for an SG in practical terms?**  
If you allow inbound on port 443, the return traffic on the ephemeral port is automatically allowed. You don't need to open outbound.

**Q41. How many SGs can be attached to an ENI and why limits matter?**  
Default 5 (max 16 with quota increase). Each adds rules evaluated per packet – too many = management complexity and negligible perf impact on Nitro.

**Q42. Can you reference one SG inside another? Why useful?**  
Yes. Example: ALB-SG → allow 443 from 0.0.0.0/0; App-SG → allow 8080 from ALB-SG. Automatically scales without IP list maintenance.

**Q43. Can SG span VPCs?**  
No, SGs are VPC-scoped. For cross-VPC, use peering + reference SG via ID (same region) or CIDR.

**Q44. Default SG vs custom SG?**  
Default SG – allows all inbound from itself, all outbound. Auto-created per VPC. Custom – you control all rules.

**Q45. How do you audit SGs in a large account?**  
- AWS Config rules (`restricted-ssh`, `vpc-sg-open-only-to-authorized-ports`)  
- Security Hub / Trusted Advisor  
- `aws ec2 describe-security-groups` + jq scripting  
- Prowler, ScoutSuite

---

## 7. KEY PAIRS & SSH

**Q46. What happens when you "lose" the `.pem` file?**  
You cannot SSH with original key. Recovery options:  
1. Use **EC2 Instance Connect** (temp key pushed via metadata) if supported.  
2. Use **SSM Session Manager** (no SSH needed, uses IAM + agent).  
3. Stop instance → detach root EBS → attach to helper instance → edit `~/.ssh/authorized_keys` → reattach.

**Q47. How to rotate SSH keys for 500 EC2 instances?**  
Production pattern:  
- SSM Run Command to push new public key / remove old  
- Or remove SSH entirely, standardize on **SSM Session Manager** + IAM  
- Immutable: new AMI with new key → ASG rollout

**Q48. Key pair types:**  
- RSA (2048-bit)  
- ED25519 (newer, shorter, faster) – not supported on all legacy AMIs  
- Stored as `.pem` (OpenSSH) or `.ppk` (PuTTY)

**Q49. What is EC2 Instance Connect?**  
Service that pushes a temporary SSH public key to instance metadata. IAM-controlled. No `.pem` needed. Requires agent on instance (Amazon Linux 2 default).

---

## 8. ELASTIC IP, ENI & NETWORKING

**Q50. What is an ENI? What can you attach/detach?**  
Virtual network card in a VPC. Has: primary private IP, secondary IPs, MAC, one EIP per IP, SGs. Can be detached and reattached to another instance – useful for failover, licensing bound to MAC.

**Q51. Primary vs Secondary ENI?**  
Primary (`eth0`) cannot be detached from instance (deleted on termination). Secondary ENIs can be moved.

**Q52. Elastic IP – billing traps:**  
- Free when associated with a running instance (one per instance).  
- Charged when: not associated, associated with stopped instance, more than one per instance.  
- Recent AWS change (2024): all public IPv4 addresses now charged.

**Q53. Public IP vs Elastic IP:**  
- Public IP – auto-assigned, changes on stop/start, free when running.  
- Elastic IP – static, you own it, portable across instances/ENIs.

**Q54. How many IPs can an instance have?**  
Depends on instance type. Formula: ENIs × IPs per ENI (both in AWS docs table). Example: m5.large = 3 ENIs × 10 IPs = 30.

**Q55. What is Enhanced Networking? ENA vs SR-IOV vs EFA?**  
- **ENA (Elastic Network Adapter)** – up to 100 Gbps, standard on modern instances.  
- **SR-IOV** – virtualization feature enabling ENA.  
- **EFA (Elastic Fabric Adapter)** – low-latency HPC/ML with OS-bypass, supports MPI/NCCL.

**Q56. What is placement of ENI for source/destination check?**  
NAT instances / custom routers need `sourceDestCheck = false` so they can forward packets not addressed to them.

**Q57. What is VPC Flow Logs for an ENI?**  
Captures metadata (src/dst IP, port, action) – ACCEPT/REJECT. Published to CloudWatch Logs / S3 / Kinesis Firehose. Critical for troubleshooting connectivity.

---

## 9. PLACEMENT GROUPS

**Q58. Three placement strategies:**  
- **Cluster** – same rack, low-latency, high-throughput (HPC). Single AZ.  
- **Spread** – each instance on different hardware. Max 7 per AZ. Critical apps needing isolation.  
- **Partition** – logical partitions on different racks. Large distributed (HDFS, Cassandra). Up to 7 partitions/AZ.

**Q59. When does a cluster PG fail?**  
If insufficient capacity on the rack. Best practice: launch all instances in single request (`RunInstances` with count).

**Q60. Can you move an existing instance into a PG?**  
Yes for stopped instances (modify instance placement). Not while running.

---

## 10. AUTO SCALING GROUP (ASG)

**Q61. What are the components of an ASG?**  
- Launch Template/Config  
- Min, Desired, Max capacity  
- Subnets (multi-AZ)  
- Health check (EC2 / ELB)  
- Scaling policies  
- Lifecycle hooks  
- Termination policies

**Q62. Types of scaling policies:**  
- **Target tracking** – maintain metric target (e.g., avg CPU 50%). Most common.  
- **Step scaling** – different actions for different alarm thresholds.  
- **Simple scaling** – single action, has cooldown.  
- **Scheduled scaling** – time-based.  
- **Predictive scaling** – ML-based forecast.

**Q63. ASG health check types?**  
- **EC2** – based on instance status check.  
- **ELB** – based on target group health (preferred for real traffic validation).  
- **VPC Lattice** / custom – newer.

**Q64. Termination policies – default order:**  
1. AZ with most instances (balance)  
2. Instance with oldest Launch Config/Template  
3. Closest to next billing hour  
4. Random  
Override: `OldestInstance`, `NewestInstance`, `OldestLaunchTemplate`, `ClosestToNextInstanceHour`, `Default`, custom.

**Q65. What is a lifecycle hook?**  
Pause instance transition at `Pending:Wait` (launch) or `Terminating:Wait` (terminate) so you can run custom logic (register in CMDB, drain connections, copy logs) before `InService` / `Terminated`.

**Q66. How does ASG rolling update work?**  
Via `Instance Refresh`:  
- Specify min healthy %, checkpoint %, warm-up  
- ASG terminates old instances in batches, launches new from updated LT  
- Compare: CloudFormation `UpdatePolicy: AutoScalingRollingUpdate`.

**Q67. Warm pools – what problem does it solve?**  
Pre-initialized stopped/hibernated instances kept ready to reduce scale-out time. Useful for apps with long boot (~5-10 min). Cheaper than keeping them running.

**Q68. Mixed Instances Policy?**  
Single ASG with multiple instance types + On-Demand/Spot percentages → resilience + cost optimization. Common pattern: 20% on-demand base + 80% spot across 4 instance types.

**Q69. Difference between Cooldown and Warm-up?**  
- **Cooldown** – pause after scaling activity before next (simple scaling only).  
- **Warm-up** – time for new instance to be ready (target tracking) so it's not counted prematurely.

**Q70. Can ASG span regions?**  
No, regional. For multi-region, use multiple ASGs + Route 53 / Global Accelerator.

---

## 11. LAUNCH TEMPLATE vs LAUNCH CONFIGURATION

**Q71. Why did AWS introduce Launch Templates?**  
LC is deprecated. LT supports:  
- Versioning  
- Multiple instance types (Mixed Instances)  
- Newer features (T3 unlimited, Capacity Reservations, Dedicated Hosts, License Manager, IMDSv2 enforcement, Elastic Inference)  
- Partial parameters (reuse in CLI/SDK)

**Q72. How to manage LT versions in production?**  
- Use `$Latest` or `$Default` alias  
- Tag versions (`v1.23-2026-04-22-golden-ami`)  
- CI/CD (Terraform/CloudFormation) creates new version, promotes to default, triggers ASG Instance Refresh.

---

## 12. LOAD BALANCERS (ELB FAMILY)

**Q73. Compare ALB / NLB / CLB / GWLB:**  
| | ALB | NLB | CLB | GWLB |
|---|---|---|---|---|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP/TLS) | 4/7 legacy | 3/4 |
| Targets | IP/Instance/Lambda/containers | IP/Instance/ALB | Instance | IP |
| Static IP | No (DNS) | Yes (1 per AZ) | No | via endpoints |
| Routing | Path, Host, Header, Query, Method | Port-based | Basic | Gateway (traffic inspection) |
| WebSockets / HTTP/2 | Yes | TLS only | Limited | – |
| Use case | Web apps, microservices | Extreme perf, static IP, non-HTTP | Legacy | 3rd-party firewalls/IDS |

**Q74. How does ALB choose a target?**  
Listener rules → target group → routing algorithm (round robin / least outstanding requests). Stickiness via cookies.

**Q75. NLB preserves source IP – why?**  
Layer 4, no termination. Target sees client IP directly – app logs client IP without X-Forwarded-For. Need to adjust SG accordingly.

**Q76. ALB health check tuning:**  
- `HealthyThreshold`, `UnhealthyThreshold`, `Interval`, `Timeout`, `Matcher`  
- Use lightweight `/healthz` endpoint (not `/` which may hit DB)

**Q77. What is cross-zone load balancing?**  
- ALB – always ON, free.  
- NLB – OFF by default, extra cost for cross-AZ data transfer.  
Enables even distribution when AZs have unequal target counts.

**Q78. What is connection draining / deregistration delay?**  
Time LB waits for in-flight requests to finish before removing target. Default 300s. Tune based on longest request.

**Q79. SSL/TLS on ALB – key options:**  
- ACM-managed certs (free, auto-renew)  
- SNI for multi-domain  
- Security policies (`ELBSecurityPolicy-TLS13-1-2-2021-06`)  
- Mutual TLS (mTLS) supported  
- End-to-end encryption: re-encrypt to target or HTTPS passthrough (NLB)

**Q80. Sticky sessions – cookie types?**  
- `AWSALB` – load balancer–generated duration  
- App-based stickiness with custom cookie  
- NLB has stickiness by source IP

---

## 13. USER DATA & INSTANCE METADATA (IMDSv1 vs v2)

**Q81. What is user data? When runs?**  
Script/cloud-init executed at first boot (by default). To re-run on every boot:  
```
#cloud-config
cloud_final_modules:
 - [scripts-user, always]
```

**Q82. Max user-data size?**  
16 KB (base64-encoded). Bigger: download from S3 in user-data.

**Q83. Instance metadata endpoint?**  
`http://169.254.169.254/latest/meta-data/`  
Get IAM role creds: `/latest/meta-data/iam/security-credentials/<role-name>`

**Q84. IMDSv1 vs IMDSv2 – why enforce v2?**  
- v1 – simple GET, vulnerable to SSRF (Capital One 2019 breach).  
- v2 – token-based, PUT to get token → GET with token header, TTL default 1 hop.  
- Enforce via LT: `HttpTokens: required`.

**Q85. IMDS hop-limit – what is it?**  
Default hop limit 1 means containers inside the instance get blocked unless limit raised to 2. Important for ECS/EKS when using IAM.

---

## 14. IAM ROLES FOR EC2

**Q86. Why use IAM roles instead of keys on EC2?**  
- No hardcoded keys  
- Auto-rotated temporary credentials (STS)  
- Revocable instantly  
- Auditable via CloudTrail

**Q87. What is an instance profile?**  
Container for an IAM role that EC2 uses. When created via console, AWS auto-creates profile with same name. CLI/Terraform – must create separately.

**Q88. How to assume a different role from an EC2 role?**  
```bash
aws sts assume-role --role-arn arn:aws:iam::ACCT:role/Target --role-session-name x
```
The EC2's role must have `sts:AssumeRole` permission and target role must trust it.

**Q89. Credential precedence on EC2?**  
1. CLI params  
2. Env vars  
3. AWS config/credential files  
4. Container creds (ECS)  
5. **Instance metadata (IAM role)** ← default  

---

## 15. PRICING MODELS

**Q90. Compare pricing models:**  
| Model | Discount | Commitment | Best for |
|---|---|---|---|
| On-Demand | 0 | None | Unpredictable / dev |
| Reserved (Standard) | up to 72% | 1/3 yr | Steady, known workload |
| Reserved (Convertible) | up to 54% | 1/3 yr | Flexible family changes |
| Savings Plan (Compute) | up to 66% | 1/3 yr | Any EC2/Fargate/Lambda |
| Savings Plan (EC2 Instance) | up to 72% | 1/3 yr | Specific family/region |
| Spot | up to 90% | None | Fault-tolerant |
| Dedicated Host | – | optional | License BYOL, compliance |
| Dedicated Instance | – | – | Same as above, less control |
| Capacity Reservation | none | none | Guarantee availability |

**Q91. Spot – how does interruption work?**  
AWS sends 2-minute interruption notice via IMDS `/latest/meta-data/spot/instance-action`. Use with interruption-tolerant apps, diversify pools, use Spot Fleet / EC2 Fleet.

**Q92. Spot allocation strategies?**  
- `lowest-price`  
- `diversified`  
- `capacity-optimized` (recommended)  
- `capacity-optimized-prioritized`  
- `price-capacity-optimized` (balance)

**Q93. Reserved Instance vs Savings Plan – which to pick?**  
- RI → fixed instance family/region/AZ, gives **capacity reservation** (zonal).  
- SP → no capacity guarantee, more flexible across services.  
- Common: mix → zonal RIs for steady critical + Compute SP for flexibility.

**Q94. Dedicated Host vs Dedicated Instance:**  
- **Dedicated Host** – physical server, you see socket/core count, BYOL (Windows/Oracle), placement control.  
- **Dedicated Instance** – runs on dedicated HW but you don't see the host.

**Q95. Capacity Reservation – difference from RI?**  
No billing discount; just guarantees capacity in an AZ. Good for DR, launch events.

---

## 16. CLOUDWATCH MONITORING & ALARMS

**Q96. Default EC2 metrics (no CWAgent):**  
CPUUtilization, DiskReadOps/WriteOps, DiskReadBytes/WriteBytes, NetworkIn/Out/PacketsIn/Out, StatusCheckFailed_System/Instance. **Memory and disk usage are NOT default.**

**Q97. How to get memory / disk metrics?**  
Install **CloudWatch Agent** (SSM can deploy). Config file defines metrics and namespace.

**Q98. Basic vs Detailed monitoring:**  
- Basic – 5-min intervals, free.  
- Detailed – 1-min intervals, paid.

**Q99. Custom metric for disk free < 20%:**  
- CWAgent reports `disk_used_percent`  
- CloudWatch Alarm → SNS → PagerDuty / Slack webhook.

**Q100. StatusCheckFailed – auto-recovery?**  
Create CloudWatch alarm → action: `recover`. Works only for system check failures on supported instance types (no instance-store, no EIP change).

---

## 17. SYSTEMS MANAGER (SSM)

**Q101. Why use SSM over SSH?**  
- No inbound port 22  
- IAM-based auth (MFA, roles, SSO)  
- Session logs to CloudWatch/S3  
- Works cross-account/region  
- Port forwarding without bastion

**Q102. Prerequisites for SSM on EC2?**  
1. SSM Agent installed (default on Amazon Linux 2, Ubuntu AMI)  
2. Instance profile with `AmazonSSMManagedInstanceCore`  
3. Network path to SSM endpoints (public or VPC endpoints for private)

**Q103. Key SSM capabilities:**  
- Session Manager (SSH replacement)  
- Run Command (mass commands)  
- Patch Manager (OS patching)  
- Parameter Store (config/secrets)  
- State Manager (desired config)  
- Automation (runbooks)  
- Inventory  
- Distributor (install software packages)

**Q104. Patch Manager baseline?**  
Approval rules defining which patches apply to which instances, based on OS, classification (Security/Critical), severity, auto-approve delay. Combined with Maintenance Window.

**Q105. Parameter Store vs Secrets Manager?**  
- Parameter Store – free (Standard), supports SecureString with KMS, basic versioning.  
- Secrets Manager – paid, auto-rotation (RDS/custom Lambda), cross-region replication.

---

## 18. EC2 IMAGE BUILDER

**Q106. Pipeline components:**  
- **Image Recipe** – base image + components (install, test).  
- **Infrastructure Configuration** – VPC/SG/instance type for build.  
- **Distribution Configuration** – accounts/regions to share.  
- **Pipeline** – schedule + triggers.

**Q107. How to enforce CIS hardening in golden AMI?**  
Use AWS-managed CIS components or build custom component YAML with shell scripts. Add vulnerability scan (Inspector).

---

## 19. LIFECYCLE: HIBERNATE / STOP / TERMINATE

**Q108. Hibernation prerequisites:**  
- Supported instance type  
- Root volume encrypted EBS ≥ RAM size  
- HIbernation enabled at launch (can't enable later)  
- Supported AMI/OS  
- Max RAM ~150 GB

**Q109. Termination protection vs Stop protection:**  
- **Termination protection** – blocks terminate via API/console.  
- **Stop protection** – blocks stop (prevents accidental data loss on ephemeral OS config).  
- Neither blocks ASG terminations unless `scale-in protection` set on instance.

**Q110. Shutdown behavior (`InstanceInitiatedShutdownBehavior`)?**  
When OS issues shutdown, instance can either `stop` (default) or `terminate`. Common gotcha: automation forgets to reset → instance vanishes.

---

## 20. SCENARIO-BASED QUESTIONS

**Q111. Your ASG scales out but new instances fail health checks – walk me through debugging.**  
1. Check ELB target health → reason column.  
2. SSM Session Manager into instance → check app logs.  
3. Verify SG allows ELB → instance port.  
4. Verify user-data executed: `cloud-init analyze show`, `/var/log/cloud-init-output.log`.  
5. Check AMI image recent patches didn't break service (e.g., Java upgrade, kernel panic).  
6. Check IAM role / instance profile present.  
7. Check /var/log/messages, dmesg for OOM.  
8. Roll back to previous LT version → `Instance Refresh`.

**Q112. Prod app running on EC2 suddenly slow – no code change. Debug.**  
- CloudWatch: CPU, memory (CWAgent), network, EBS IOPS/throughput, burst balance.  
- Neighbor noisy? If on `t` family – CPU credit exhaustion (`CPUCreditBalance=0`).  
- EBS → check `VolumeQueueLength`, `VolumeThroughputPercentage`.  
- Application: `top`, `iotop`, `vmstat`, `pidstat`.  
- Check recent patches via SSM Inventory.  
- Check VPC Flow Logs for retransmits / drops.  
- If DB call slow: RDS Perf Insights.

**Q113. Your Jenkins on EC2 lost SSH access after kernel upgrade – recover.**  
1. Try SSM Session Manager.  
2. Try EC2 Serial Console (if enabled at account + instance type supported).  
3. Stop instance → detach root EBS → attach to rescue instance → mount → fix `/boot/grub/grub.cfg` or `/etc/fstab` → reattach → start.

**Q114. You need to migrate 300 EC2 instances to gp3. Zero downtime. Approach?**  
1. Audit volumes: `describe-volumes` filter gp2.  
2. Test in lower env: `modify-volume --volume-type gp3`.  
3. Write Lambda / SSM Automation that iterates volumes in batches (respect modification cool-down 6h per volume).  
4. Validate `ModificationState`.  
5. Update Terraform state / LT to gp3 so newly launched match.  
6. Cost savings tracked via CUR.

**Q115. Website intermittently times out only for some users. EC2 behind ALB. How to diagnose?**  
- ALB access logs to S3 → query with Athena; filter 5xx / high `target_processing_time`.  
- Check target group health flaps.  
- If AZ-specific → cross-zone LB off + target skew.  
- Check sticky session causing hot instance.  
- Path-specific? ALB rules.  
- CloudFront / DNS caching stale.

**Q116. You must meet a compliance rule: "No instance without encrypted EBS."**  
- Enable account-wide default EBS encryption.  
- AWS Config rule: `encrypted-volumes`.  
- Auto-remediate via SSM Automation: snapshot → copy encrypted → create volume → swap.  
- Service Control Policy (SCP) to deny creating unencrypted volume.

**Q117. Developer asks for 1 TB RAM for ML job for 6 hours – how to provision cost-effectively?**  
- Spot `r6i.32xlarge` / `x2idn.32xlarge` with interruption checkpointing, or  
- On-Demand Capacity Reservation + Savings Plan margin.  
- Terminate after job (use lifecycle hook + TTL tag + Lambda cleanup).

**Q118. Black Friday spike anticipated. Prepare ASG.**  
- Predictive scaling + scheduled warm-up 30 min before peak.  
- Warm pool of 50 pre-baked instances.  
- Capacity Reservations in each AZ.  
- Pre-scale to min=50% of expected peak.  
- Fail-fast deploys disabled during freeze window.

**Q119. One EC2 in a cluster always hits 100% CPU while others are idle.**  
- Sticky session misconfiguration (one user / stuck connection).  
- ALB least-outstanding-requests might help.  
- Check if instance has crashed thread loop (`top -H`, `jstack`).  
- NLB source-IP hash leading to skew.

**Q120. You must restrict SSH access to only office IPs but office IP changes monthly.**  
- Use prefix list (managed) + update via automation.  
- Better: replace SSH with SSM Session Manager + IAM.  
- Or ZTNA (AWS Verified Access).

---

## 21. TROUBLESHOOTING QUESTIONS

**Q121. Instance launch fails with `InsufficientInstanceCapacity` – what does it mean and fix?**  
AWS doesn't have capacity for that type in that AZ. Fix: try another AZ, smaller instance, different family, or use Capacity Reservation.

**Q122. `InstanceLimitExceeded` error:**  
Account vCPU quota hit. Request quota increase via Service Quotas (on-demand vCPU per family, spot vCPU separate).

**Q123. Cannot SSH – troubleshooting checklist:**  
1. Instance `running`, status checks 2/2?  
2. SG allows 22 from your IP?  
3. NACL allows inbound/outbound 22 and ephemeral?  
4. Subnet has IGW route if public? EIP attached?  
5. Correct key, correct user (`ec2-user`, `ubuntu`, `admin`)?  
6. Key file perms `chmod 400`?  
7. sshd running? (EC2 Serial Console / SSM)  
8. `/etc/ssh/sshd_config` not denying?  
9. Host-based firewall (firewalld/iptables) blocking?  
10. `/home/ec2-user/.ssh/authorized_keys` perms?

**Q124. Web app returns 502 Bad Gateway through ALB.**  
- Target unhealthy or TCP reset.  
- Idle timeout mismatch: ALB 60s default, target shorter → premature close.  
- Incorrect listener protocol (HTTPS→HTTP target).  
- Keep-alive not enabled on target.  
- Check ALB access logs `elb_status_code=502` `target_status_code=-`.

**Q125. `Cannot create volume` – KMS errors.**  
IAM role missing `kms:CreateGrant`, `kms:DescribeKey`, `kms:GenerateDataKeyWithoutPlaintext`. Check KMS key policy allows role.

**Q126. EBS performance suddenly drops in a dev gp2 volume:**  
Burst balance exhausted. `BurstBalance=0`. Fix: migrate to gp3 or provision larger gp2.

**Q127. Application logs show `too many open files`:**  
- `ulimit -n` too low (default 1024).  
- Fix: `/etc/security/limits.conf`, systemd `LimitNOFILE=65536`.  
- Bake into AMI / user-data.

**Q128. EC2 CPU Steal Time is 30% – what does it mean?**  
Hypervisor taking CPU away (noisy neighbor). Common on `t` burst family. Mitigation: move to non-burst family (m/c/r) or dedicated host.

**Q129. `NoSpaceLeftOnDevice` but `df` shows free space:**  
- Inodes exhausted: `df -i`. Fix: delete small files, recreate FS, use xfs.

**Q130. `Request limit exceeded` from AWS API on EC2 automation:**  
API throttling. Exponential backoff, use paginators, switch to event-driven (EventBridge) or CloudTrail → S3 batch.

**Q131. Instance has no disk I/O but CPU high:**  
Possibly crypto mining malware (common on exposed EC2). Isolate SG → snapshot → forensics → rotate keys → redeploy.

**Q132. User-data did not execute on relaunch:**  
Cloud-init runs once. Check `/var/lib/cloud/instance` semaphore. For re-run: `cloud-init clean && reboot` or `cloud_final_modules: [[scripts-user, always]]`.

**Q133. Lost EIP after reboot – but reboot shouldn't change IP?**  
Likely EIP wasn't actually associated (auto-assigned public IP). Assign EIP explicitly.

**Q134. ASG keeps launching and terminating instances (flapping):**  
Health check fails during long boot → ASG kills. Fix: increase `HealthCheckGracePeriod`, optimize boot.

**Q135. `CredentialsNotFound` from SDK on EC2:**  
Instance profile not attached or role trust policy wrong. `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/` should show role.

**Q136. `ModifyVolume` returns `IncorrectModificationState`:**  
Volume still optimizing from previous change (6-hour cool-down). Wait.

**Q137. SSH works from laptop but fails from Jenkins agent:**  
SG differs, NAT IP, outdated key, TCP timeouts, MTU mismatch over VPN.

**Q138. Instance reboots randomly:**  
- System status check → AWS hardware fault.  
- Kernel panic → `/var/log/dmesg`.  
- OOM → `dmesg | grep -i oom`.  
- ECC memory errors.  
- Scheduled maintenance event: `describe-instance-status --include-all`.

---

## 22. REAL-TIME PRODUCTION ISSUES (LinkedIn-style)

**Q139. In production we saw ALB 504 Gateway Timeout spiking for 5 minutes. What would your first 3 commands be?**  
1. `aws elbv2 describe-target-health` for the TG.  
2. Athena query on ALB access logs for that window, group by `target_processing_time > 60`.  
3. CloudWatch: TargetResponseTime P95 vs request count, correlate with deployment timeline.

**Q140. A team member accidentally terminated a prod EC2. How do you recover?**  
- If AMI and LT existed → relaunch.  
- If root EBS `DeleteOnTermination=false` → attach to new instance.  
- Else: restore latest EBS snapshot.  
- Post-mortem: enable termination protection, IAM SCP, CloudTrail alert on `TerminateInstances`.

**Q141. How do you patch 2,000 EC2 instances with zero downtime?**  
- Organize via tags (env, app, wave).  
- SSM Patch Manager + Maintenance Windows per wave.  
- ASG with rolling Instance Refresh using new golden AMI (immutable).  
- Pre/post hooks to drain LB, notify.  
- Blue/green for stateful.

**Q142. Finance asks: "Why EC2 cost jumped 30% last month?"**  
- Cost Explorer group by Linked Account, Usage Type, Tag.  
- CUR in Athena for hourly.  
- Detect: new NAT Gateway data, unencrypted snapshots aging, EIP unattached, orphan EBS, idle instances.  
- Trusted Advisor / Compute Optimizer recommendations.

**Q143. Security team reports: instance communicating with known malicious IP. Respond.**  
1. GuardDuty finding investigate.  
2. Isolate: new restrictive SG (only forensics SG).  
3. Snapshot memory (if possible) + EBS.  
4. Revoke IAM role.  
5. Terminate compromised.  
6. Rotate any exposed creds.  
7. Root-cause: SSM Inventory / CloudTrail / VPC Flow Logs.

**Q144. A Node.js app on EC2 crashes every 6 hours with OOM despite 16 GB RAM.**  
- Heap snapshot: `--inspect` + Chrome DevTools.  
- Check native mem (Buffers, WASM).  
- `cat /sys/fs/cgroup/memory.stat`.  
- Set `--max-old-space-size`, `NODE_OPTIONS`.  
- Enable swap as safety net (not for prod, temporary).

**Q145. You need to bake an AMI with Docker + Jenkins agent + custom certificates. Reproducible pipeline. Design.**  
- Terraform: Image Builder pipeline + components (install Docker, install agent, copy certs from SSM Parameter Store), infra config (isolated subnet), distribution config (dev/stage/prod accounts).  
- Trigger on upstream Amazon Linux 2023 AMI update via EventBridge.  
- Scan with Inspector before distribute.  
- Version tagged; LT updated in CI.

**Q146. A team insists on `.pem` file shared in Git for CI. How to refuse politely and fix?**  
- Use SSM Session Manager or Instance Connect (ephemeral keys).  
- If SSH mandatory → HashiCorp Vault SSH CA or AWS SSM RunCommand instead of SSH.  
- CI: OIDC federation to AWS (no long-lived keys at all).

**Q147. Dev deployed wrong AMI id to prod ASG and 30% traffic 500s. Roll back.**  
`aws autoscaling start-instance-refresh` with previous LT version. Or set `desired=0` → `desired=N` with old LT if urgent. Check CodeDeploy / Jenkins pipeline lock.

**Q148. Latency between EC2 in us-east-1a and RDS in us-east-1b is 3 ms – acceptable?**  
Yes, cross-AZ ~1-3 ms normal. Issue if app is chatty (1000 sequential queries). Solution: read replica in same AZ, ProxySQL, batch queries, connection pool (RDS Proxy).

**Q149. You want to SSH to 200 instances to grep logs. Best way?**  
SSM Run Command with target tags + output to S3. Alternative: centralized logging (CloudWatch Logs, ELK, OpenSearch) – don't SSH.

**Q150. HR: "Can we deploy EC2 without any public IP?"**  
Yes: private subnet + NAT Gateway for outbound; SSM + VPC Endpoints (no IGW); ALB internal / public-facing with private targets.

---

## 23. DEVOPS-FOCUSED EC2 QUESTIONS (Terraform / Ansible / Jenkins / K8s)

**Q151. Terraform – why prefer `aws_launch_template` over `aws_launch_configuration`?**  
LC deprecated. LT supports versions (`create_before_destroy`), newer features, mixed instances in ASG resource.

**Q152. Terraform pitfall: Changing user_data forces replacement. How to avoid?**  
Use Launch Template with version bump; ASG Instance Refresh triggered outside TF or via `lifecycle { create_before_destroy = true }` on LT version.

**Q153. Terraform: manage ASG desired_capacity drift:**  
`lifecycle { ignore_changes = [desired_capacity] }` – since scaling policies change it at runtime.

**Q154. Ansible dynamic inventory for EC2:**  
`aws_ec2` plugin in `inventory.aws_ec2.yml`, filter by tag, groups by tag. Needs IAM read access.

**Q155. How Jenkins on EC2 should talk to AWS APIs:**  
IAM instance profile with needed perms. Use Jenkins EC2 Fleet plugin for dynamic agents as Spot in ASG.

**Q156. Kubernetes on EC2 (EKS self-managed nodegroup):**  
- Bootstrap via user-data: `/etc/eks/bootstrap.sh <cluster>`.  
- CNI (VPC CNI) consumes secondary IPs from ENI – plan subnet CIDR.  
- Node IAM role needs `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`.  
- IMDSv2 hop-limit 2 if pods use IAM.  
- Karpenter vs Cluster Autoscaler: Karpenter provisions raw EC2 without ASG.

**Q157. EKS pod can't reach IMDS for IAM:**  
Use IRSA (IAM Roles for Service Accounts) instead of node role. Restrict IMDS at node via `HttpPutResponseHopLimit=1`, `HttpTokens=required`.

**Q158. How to get consistent hostname on launch?**  
User-data: `hostnamectl set-hostname $(curl -s http://169.254.169.254/latest/meta-data/instance-id).ec2.internal`.  
Or Terraform `tags = { Name = "app-${count.index}" }` + set on boot.

**Q159. CI/CD pipeline for immutable EC2 golden-AMI rollout:**  
1. Code merge → Jenkins build.  
2. Image Builder pipeline triggered (or Packer).  
3. AMI tested (Inspector, Goss/InSpec).  
4. AMI tagged `release=vX.Y.Z`.  
5. Terraform updates LT to new AMI (new version).  
6. ASG Instance Refresh with canary %.  
7. CloudWatch synthetics validate → full rollout or rollback.

**Q160. Docker on EC2 vs ECS vs EKS:**  
- Plain Docker EC2 – full control, manual ops.  
- ECS (EC2 launch type) – AWS-managed scheduler.  
- ECS Fargate – no EC2.  
- EKS – Kubernetes API, more ecosystem.  
- Criteria: team skills, portability, scale, cost.

---

## BONUS: 20 RAPID-FIRE ONE-LINERS (often opened with)

1. Max IPs per ENI on m5.large? → 10  
2. Default root volume size Amazon Linux 2023? → 8 GB  
3. Can you enable IMDSv2 on running instance? → Yes, `modify-instance-metadata-options`.  
4. Can EBS be moved across AZs? → Only via snapshot.  
5. Can you convert instance store-backed AMI to EBS? → Yes via registration of new AMI after migrating root.  
6. Default tenancy? → `default` (shared).  
7. What's `T2 unlimited`? → Burst beyond credits with extra charge.  
8. Maximum ENIs on c5n.18xlarge? → 15.  
9. Can NLB terminate TLS? → Yes (since 2019).  
10. Can ALB have static IP? → Not directly; use Global Accelerator or NLB fronting ALB.  
11. What port does SSM Agent use? → 443 outbound to endpoints.  
12. Are spot instances billed per second? → Yes (Linux, ≥1 min).  
13. Can you stop a spot instance? → Only if launched via persistent request.  
14. What is the EC2 API version of "restart"? → `RebootInstances`.  
15. Does reboot incur billing break? → No (considered continuous).  
16. Max EBS per instance? → 28 (block devices, varies by type).  
17. Root device types? → `ebs` or `instance-store`.  
18. Min EBS size for gp3? → 1 GiB.  
19. Can you run Windows on Graviton? → Yes (Windows on Arm previews; limited).  
20. Max Spot price you should bid? → On-demand price (default). Bidding higher doesn't accelerate; AWS bills the market price.

---

## SUGGESTED PREP STRATEGY (3–4 YRS EXP)

1. **Week 1** – Fundamentals + EBS + Networking (Q1–Q60).  
2. **Week 2** – ASG + LB + SSM + IAM (Q61–Q110).  
3. **Week 3** – Scenario + Troubleshooting (Q111–Q138).  
4. **Week 4** – Real-time prod + DevOps tooling (Q139–Q160) + hands-on labs.  
5. Practice whiteboarding 3 scenarios per day.  
6. Build one full pipeline: Terraform → Image Builder → ASG → ALB → CloudWatch → SSM patching.  
7. Always finish the answer with: *cost, security, observability, resilience, automation* perspectives – MNC interviewers love multi-dimensional answers.

---

## COMMON FOLLOW-UP "WHY" QUESTIONS TO PRACTICE

- Why did AWS deprecate Launch Configuration?  
- Why is IMDSv2 important – walk through the SSRF attack vector.  
- Why is EBS multi-attach rarely recommended?  
- Why are Spot + Stateful workloads dangerous?  
- Why might detailed monitoring be cheaper overall?  
- Why should production never use the default VPC / default SG?  
- Why is the `t` family not ideal for prod baseline-critical apps?

---

*End of Document – 160+ unique questions across all EC2 modules. Good luck with your interviews!*
